---
title: Reduce Your Testcases with Bugpoint and Custom Scripts
url: https://blog.llvm.org/2015/11/reduce-your-testcases-with-bugpoint-and.html
published: "2015-11-12T20:18:00Z"
feed: llvm
guid: https://blog.llvm.org/2015/11/reduce-your-testcases-with-bugpoint-and.html
---

# Reduce Your Testcases with Bugpoint and Custom Scripts

LLVM provides many useful command line tools to handle bitcode: _opt_ is the most widely known and is used to run individual passes on an IR module, and _llc_ invokes the backend to generate an assembly or object file from an IR module. Less known but very powerful is _bugpoint_, the automatic test case reduction tool, that should be part of every developer's toolbox.

The _bugpoint_ tool helps to reduce an input IR file while preserving some interesting behavior, usually a compiler crash or a miscompile. Multiple strategies are involved in the reduction of the test case (shuffling instructions, modifying the control flow, etc.), but because it is oblivious to the LLVM passes and the individual backend specificities, "it may appear to do stupid things or miss obvious simplifications", as stated in the [official description](http://llvm.org/docs/CommandGuide/bugpoint.html#description). The [documentation](http://llvm.org/docs/Bugpoint.html) gives some insights on the strategies that can be involved by _bugpoint_, but the details are beyond the scope of this post.

Read on to learn how you can use the power of bugpoint to solve some non-obvious problems.

## Bugpoint Interface Considered Harmful

Bugpoint is a powerful tool to reduce your test case, but its interface can lead to frustration (as stated in the documentation: " _bugpoint can be a remarkably useful tool, but it sometimes works in non-obvious ways_"). One of the main issue seems to be that _bugpoint_ is ironically too advanced! It operates under three modes and switches automatically among them to solve different kind of problem: crash, miscompilation, or code generation (see [the documentation](http://llvm.org/docs/Bugpoint.html) for more information on these modes). However it is not always obvious to know beforehand which mode will be activated and which strategy _bugpoint_ is actually using.

I found that for most of my uses, I don't want the _advanced_ _bugpoint_ features that deal with pass ordering for example, and I don't need _bugpoint_ to detect which mode to operate and switch automatically. For most of my usage, the \`compile-custom\` option is perfectly adequate: similar to

\`git bisect\`, it allows you to provide a script to _bugpoint_. This script is a black box for _bugpoint_, it needs to accept a single argument (the bitcode file to process) and needs to return 0 if the bitcode does not exhibit the behavior you're interested in, or a non zero value in the other case. _Bugpoint_ will apply multiple strategies in order to reduce the test case, and will call your custom script after each transformation to validate if the behavior you're looking for is still exhibited. The invocation for _bugpoint_ is the following:

$ ./bin/bugpoint -compile-custom -compile-command=./check.sh -opt-command=./bin/opt my\_test\_case.ll

The important part is the two options _-compile-custom_ and _-compile-command=path\_to\_script.sh_ that indicate to _bugpoint_ that it should use your own script to process the file. The other important part is the _-opt-command_ option that should point to the correct _opt_ that will be used to reduce the test case. Indeed by default _bugpoint_ will search in the path for _opt_ and may use an old system one that won't be able to process your IR properly, leading to some curious error message:

\\*\\*\\* Debugging code generator crash!

Checking for crash with only these blocks:  diamond .preheader .lr.ph .end: error: Invalid type for value

simplifycfg failed!

Considering such a script \`check.sh\`, running it with your original test case this way:

$ ./check.sh my\_test\_case.ll && echo "NON-INTERESTING" \|\| echo "INTERESTING"

should display INTERESTING before you try to use it with bugpoint, or you may very well be surprised. In fact _bugpoint_ considers the script as a compile command. If you start with an NON-INTERESTING test case and feed it to _bugpoint_, it will assume that the code compiles correctly, and will try to assemble it, link it, and execute it to get a reference result. This is where _bugpoint_ behavior can be confusing when it automatically switches mode, leaving the user with a confusing trace. A correct invocation should lead to a trace such as:

./bin/bugpoint  -compile-custom  -compile-command=./check.sh  -opt-command=./bin/opt slp.ll

Read input file      : 'slp.ll'

\\*\\*\\* All input ok

Initializing execution environment: Found command in: ./check.sh

Running the code generator to test for a crash:

Error running tool:

./check.sh bugpoint-test-program-1aa0e1d.bc

\\*\\*\\* Debugging code generator crash!

Checking to see if we can delete global inits: <crash>

\\*\\*\\* Able to remove all global initializers!

Checking for crash with only these blocks:    .lr.ph6.preheader .preheader .lr.ph.preheader .lr.ph .backedge  .\_crit\_edge.loopexit... <11 total>: <crash>

Checking for crash with only these blocks: .preheader .backedge .lr.ph6.preheader:

Checking for crash with only these blocks: .lr.ph .\_crit\_edge:

...

...

Checking instruction:   store i8 %16, i8\* getelementptr inbounds (\[32 x i8\], \[32 x i8\]\* @cle, i64 0, i64 15), align 1, !tbaa !2

\\*\\*\\* Attempting to perform final cleanups: <crash>

Emitted bitcode to 'bugpoint-reduced-simplified.bc'

In practice the ability to write a custom script is very powerful, I will go over a few use cases I recently used _bugpoint_ with.

## Search For a String in the Output

I recently submitted a patch (http://reviews.llvm.org/D14364) for a case where the loop vectorizer didn't kick-in on a quite simple test case. After fixing the underlying issue I needed to submit a test with my patch. The original IR was a few hundred lines. Since I believe it is good practice to reduce test cases as much as possible, bugpoint is often my best friend. In this case the analysis result indicates "Memory dependences are safe with run-time checks" on the output after my patch.

Having compiled \`opt\` with and without my patch and copied each version in \`/tmp/\` I wrote this shell script:

#!/bin/bash

/tmp/opt.original -loop-accesses-analyze$1 \| grep"Memory dependences are safe"

res\_original=$?

/tmp/opt.patched -loop-accesses-analyze$1 \| grep"Memory dependences are safe"

res\_patched=$?

\[\[$res\_original==1&&$res\_patched==0\]\] && exit1

exit0

It first runs the bitcode supplied as argument to the script (the $1 above) through _opt_ and uses _grep_ to check for the presence of the expected string in the output. When _grep_ exits, _$?_ contains with 1 if the string is **not** present in the output. The reduced test case is valid if the original _opt_ didn't produce the expected analysis but the new _opt_ did.

## Reduce While a Transformation Makes Effects

In another case (http://reviews.llvm.org/D13996), I patched the SLP vectorizer and I wanted to reduce the test case so that it didn't vectorize before my changes but vectorizes after:

#!/bin/bash

set-e

/tmp/opt.original -slp-vectorizer-S> /tmp/original.ll $1

/tmp/opt.patched -slp-vectorizer-S> /tmp/patched.ll $1

diff /tmp/original.ll /tmp/patched.ll && exit0

exit1

The use of a custom script offers flexibility and allows to run any complex logic to decide if a reduction is valid or not. I used it in the past to reduce crashes on a specific assertion and avoiding the reduction leading to a different crash, or to reduce for tracking instruction count regressions or any other metric.

## Just Use FileCheck

LLVM comes with a [Flexible pattern matching file verifier](http://llvm.org/docs/CommandGuide/FileCheck.html) (FileCheck) that the tests are using intensively. You can annotate your original test case and write a script that reduce it for your patch. Let's take an example from the public LLVM repository with commit r252051 _"_\[SimplifyCFG\] Merge conditional stores _"._ The associated test in the validation is test/Transforms/SimplifyCFG/merge-cond-stores.ll ; and it already contains all the check we need, let's try to reduce it. For this purpose you'll need to process one function at a time, or bugpoint may not produce what you expect: because the check will fail for one function, bugpoint can do any transformation to another function and the test would still be considered "interesting". Let's extract the function test\_diamond\_simple from the original file:

$ ./bin/llvm-extract -func=test\_diamond\_simple test/Transforms/SimplifyCFG/merge-cond-stores.ll -S > /tmp/my\_test\_case.ll

Then checkout and compile _opt_ for revision r252050 and r252051, and copy them in /tmp/opt.r252050 and /tmp/opt.r252051. The check.sh script is then based on the _CHECK_ line in the original test case:

#!/bin/bash

\# Process the test before the patch and check with FileCheck,

\# this is expected to fail.

/tmp/opt.r252050 -simplifycfg-instcombine-phi-node-folding-threshold=2-S<$1 \| ./bin/FileCheck merge-cons-stores.ll

original=$?

\# Process the test after the patch and check with FileCheck,

\# this is expected to succeed.

/tmp/opt.r252051 -simplifycfg-instcombine-phi-node-folding-threshold=2-S<$1 \| ./bin/FileCheck merge-cons-stores.ll

patched=$?

\# The test is interesting if FileCheck failed before and

\# succeed after the patch.

\[\[$original!=0&&$patched==0\]\] && exit1

exit0

I intentionally selected a very well written test to show you both the power of bugpoint and its limitation. If you look at the function we just extracted in _my\_test\_case.ll_ for instance:

; CHECK-LABEL: @test\_diamond\_simple

; This should get if-converted.

; CHECK: store

; CHECK-NOT: store

; CHECK: ret

definei32@test\_diamond\_simple(i32\* %p, i32\* %q, i32%a, i32%b) {

entry:

%x1 = icmpeqi32%a, 0

bri1%x1, label%no1, label%yes1

yes1:

storei320, i32\* %p

brlabel%fallthrough

no1:

%z1 = addi32%a, %b

brlabel%fallthrough

fallthrough:

%z2 = phii32 \[ %z1, %no1 \], \[ 0, %yes1 \]

%x2 = icmpeqi32%b, 0

bri1%x2, label%no2, label%yes2

yes2:

storei321, i32\* %p

brlabel%end

no2:

%z3 = subi32%z2, %b

brlabel%end

end:

%z4 = phii32 \[ %z3, %no2 \], \[ 3, %yes2 \]

reti32%z4

}

The transformation introduced in this patch allows to merge the stores in the true branches yes1 and yes2:

declarevoid@f()

definei32@test\_diamond\_simple(i32\* %p, i32\* %q, i32%a, i32%b) {

entry:

%x1 = icmpeqi32%a, 0

%z1 = addi32%a, %b

%z2 = selecti1%x1, i32%z1, i320

%x2 = icmpeqi32%b, 0

%z3 = subi32%z2, %b

%z4 = selecti1%x2, i32%z3, i323

%0 = ori32%a, %b

%1 = icmpeqi32%0, 0

bri1%1, label%3, label%2

; <label>:2 ; preds = %entry

%simplifycfg.merge = selecti1%x2, i32%z2, i321

storei32%simplifycfg.merge, i32\* %p, align4

brlabel%3

; <label>:3 ; preds = %entry, %2

reti32%z4

}

The original code seems pretty minimal, the variable and block names are explicit, it is easy to follow and you probably wouldn't think about reducing it. For the exercise, let's have a look at what bugpoint can do for us here:

definevoid@test\_diamond\_simple(i32\* %p, i32%b) {

entry:

bri1undef, label%fallthrough, label%yes1

yes1:; preds = %entry

storei320, i32\* %p

brlabel%fallthrough

fallthrough:; preds = %yes1, %entry

%x2 = icmpeqi32%b, 0

bri1%x2, label%end, label%yes2

yes2:; preds = %fallthrough

storei321, i32\* %p

brlabel%end

yes2:; preds = %yes2, %fallthrough

retvoid

}

Bugpoint figured out that the _no_ branches were useless for this test and removed them. The drawback is that bugpoint also has a tendency to introduce _undef_ or _unreachable_ here and there, which can make the test more fragile and harder to understand.

## Not There Yet: Manual Cleanup

At the end of the reduction, the test is small but probably not ready to be submitted with your patch "as is". Some cleanup is probably still needed: for instance bugpoint won't convert invoke into calls,  remove metadata, tbaa informations, personality function, etc. We also saw before that bugpoint can modify your test in unexpected way, adding _undef_ or _unreachable_. Also you probably want to rename the variables to end up with a readable test case.

Fortunately, having the _check.sh_ script at hand is helpful in this process, since you can just manually modify your test and run continuously the same command:

$ ./check.sh my\_test\_case.ll && echo "NON-INTERESTING" \|\| echo "INTERESTING"

While the result is  INTERESTING you know you keep having a valid test and you can continue to proceed with your cleanup.

Keep in mind that bugpoint can do far more, but hopefully this subset will be helpful to the ones that are still struggling with its command line options.

Finally, I'm grateful to Manman Ren for her review of this post.
