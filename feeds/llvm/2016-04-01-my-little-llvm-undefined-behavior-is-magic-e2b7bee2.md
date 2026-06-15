---
title: 'My Little LLVM: Undefined Behavior is Magic!'
url: https://blog.llvm.org/2016/04/undefined-behavior-is-magic.html
published: "2016-04-01T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/2016/04/undefined-behavior-is-magic.html
---

# My Little LLVM: Undefined Behavior is Magic!

[![](https://2.bp.blogspot.com/-uilL1GOdl0E/WrVbVxsAxHI/AAAAAAAAAoU/oDi-ww1rx8I-xlHhmFHtUiLK_FgCUVajQCLcBGAs/s640/DragonPony.png)](https://2.bp.blogspot.com/-uilL1GOdl0E/WrVbVxsAxHI/AAAAAAAAAoU/oDi-ww1rx8I-xlHhmFHtUiLK_FgCUVajQCLcBGAs/s1600/DragonPony.png)

There’s been [lots of discussion online](https://www.google.com/#q=%22undefined+behavior%22+%22should+just%22+OR+%22just+work%22+site:news.ycombinator.com) ( [and](http://www.complang.tuwien.ac.at/kps2015/proceedings/KPS_2015_submission_29.pdf) [then](https://gcc.gnu.org/ml/gcc/2016-02/msg00381.html) [quite](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html) [some](http://blog.regehr.org/archives/761) [more](https://groups.google.com/forum/m/#!msg/boring-crypto/48qa1kWignU/o8GGp2K1DAAJ)) about compilers abusing undefined behavior. As a response the LLVM compiler infrastructure is rebranding and adopting a motto to make undefined behavior friendlier and less prone to [corruption](https://youtu.be/H6g15JroPow?t=1m32s).

The re-branding puts to rest a long-standing issue with LLVM’s “dragon” logo [actually being a wyvern](http://scifi.stackexchange.com/questions/34088/differences-between-dragon-drake-wyrm-and-wyvern) with an [upside-down head](https://twitter.com/siracusa/status/659741883166547968), a special form of undefined behavior in its own right. The logo is now clearly a pegasus pony.

Another great side-effect of this rebranding is increased security by auto-magically closing all vulnerabilities used by the hacker who goes by the pseudonym “ [Pinkie Pie](https://www.google.com/search?q=pinkie%20pie%20hacker)”.

These new features are enabled with the -rainbow clang option, in honor of Rainbow Dash’s unary name.

A Few Examples

C++’s memory model specifies that data races are undefined behavior. It is well established that [no sane compiler would optimize atomics](http://wg21.link/n4455), LLVM will therefore supplement the Standard’s happens-before relationship with an LLVM-specific happens-to-work relationship. On most architectures this will be implemented with micro-pause primitives such as x86’s rep rep rep nop instruction.

Shifts by bit-width or larger will now return a normally-distributed random number. This also obsoletes rand() and std::random\_shuffle.

bool now obeys the rules of [truthiness](https://en.wikipedia.org/wiki/Truthiness) to avoid that annoying “but what if it’s not zero or one?” interview question. Further, incrementing a bool with ++ now does the right thing.

Atomic integer arithmetic is already specified to be two’s complement. Regular arithmetic will therefore now also be atomic. Except when volatile, but not when volatile atomic.

[NaNs](http://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html#899) will now compare equal, subnormals are free to self-classify as normal / zero / other, negative zero simply won’t be a thing, IEEE-754 has been upgraded to PONY-754, floats will still [round with style](http://en.cppreference.com/w/cpp/types/numeric_limits/float_round_style), and generating a signaling NaN is now guaranteed to not be quiet by being equivalent to putchar('\\a'). While we’re at it none of math.h will set errno anymore. This has nothing to do with undefined behavior but seriously, errno?

Type-punning isn’t a thing anymore. We’re renaming it to type-pony-ing, but it doesn’t do anything surprising besides throw parties. AND WHO DOESN’T LIKE PARTIES‽ EVEN SECURITY PEOPLE DO! 🎉

## A Word From Our Sponsors

The sanitizers—especially [undefined behavior sanitizer](http://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html), [address sanitizer](http://clang.llvm.org/docs/AddressSanitizer.html) and [thread sanitizer](http://clang.llvm.org/docs/ThreadSanitizer.html)—are great tools when dealing with undefined behavior. Use them on your tests, combine them with [fuzzers](http://llvm.org/docs/LibFuzzer.html), try them as cupcake topping! Be warned: their runtimes aren’t designed to be secure and you shouldn’t ship them in production code!

## Cutie Marks

To address the [horse](https://twitter.com/horse_clang) in the room: we’ve left the new LLVM logo’s cutie mark as implementation-defined. Different instances of the logo can use their own cutie mark to illustrate their proclivities, but must clearly document them.

_Posted by [JF Bastien](https://twitter.com/jfbastien) and [Michael Spencer](https://twitter.com/Bigcheesegs)._
