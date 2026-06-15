---
title: LLVM relicensing update & call for help
url: https://blog.llvm.org/posts/2021-11-18-relicensing-update/
published: "2021-11-18T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2021-11-18-relicensing-update/
---

# LLVM relicensing update & call for help

In this blog post, I’d like to summarize the main points I talked about in therelicensing update presentation at the 2021 LLVM Developer’s meeting.

The very short summary is that we are currently in the long tail phase ofcollecting relicensing agreements of past contributors. We already at the timeof this writing have more than 94% of older code relicensed. We hope to crowdsource getting through the long tail to get as close to 100% as possible.

# Call for help

You can help by looking through [the list of remaining individuals andcorporations we need to get agreementsfrom](https://docs.google.com/spreadsheets/d/18_0Hog_eSwES8lKwf7WJal3yBwwcYfvPu1yCfZnTcek/edit?usp=sharing),and reaching out to them, or letting us know at [license-questions@llvm.org](mailto:license-questions@llvm.org) howwe can reach out to them. Also, if you’d happen to have any other information that you think could help us, please do let us know at [license-questions@llvm.org](mailto:license-questions@llvm.org).

More detailed guidance on how you can help are available at [https://foundation.llvm.org/docs/relicensing\_long\_tail](https://foundation.llvm.org/docs/relicensing_long_tail).

In the rest of this blog post, I’m going to give a bit more historicalbackground and describe the current status in more detail.

# The LLVM relicensing effort: phases over time

The relicensing effort started some time ago. Let me first very briefly describethe various phases before going in more detail. In the years leading up to 2015,it became clear that there were some issues with the license LLVM had at thetime.

A different license was the answer to those issues. Between roughly 2015 and2017 the focus was on deciding what different license would be best suited.

Once it was decided what the new license was going to be, we started working ongetting all code to be covered by this new license. That includes gettingagreement from all copyright holders of existing code to share their pastcontributions under the new license. As you can see on the timeline, gettingthose agreements is the current focus of the relicensing effort.

Maybe we won’t be able to get an agreement for every single past contribution.In that case, we have a number of options to get to the point where we can claimthat all code in the LLVM project is covered by the new LLVM license. We callthis phase of relicensing “the end game”.

![timeline](https://blog.llvm.org/img/relicensing2021/timeline.png)

# Why relicense?

The old license consists of 3 components: the UIUC license that covered all thecode, the MIT license that additionally covered the code in run-time libraries,and a few sentences on granting patent rights in the developer policy.

This caused [the following 3 issues](https://lists.llvm.org/pipermail/llvm-dev/2015-October/091536.html):

1. Some were blocked from contributing because of the text in the patent sectionin the developer policy. The wording could be interpreted as requiring givingunnecessarily broad access to patent rights when contributing to LLVM. Thatmade it impossible for some companies to contribute.

2. The run time libraries were dual licensed under the UIUC and MIT license; therest of the code only under the UIUC license. Therefore, we could not easilymove code to run time libraries from other parts. The reason run timelibraries were dual licensed was to enable linking to run time librarybinaries without requiring attribution to LLVM.

3. The wording on patent rights in the developer policy was fuzzy and imprecise,leading to uncertainty over whether it did provide the intended protection.


# A new license

After exploring a range of options, it was decided that the best solution to solve these issues was to have all code licensed under the [Apache-2.0 with LLVM exception](https://foundation.llvm.org/relicensing/LICENSE.txt) license:

- Apache-2.0 contains well-understood patent granting which addresses the firstand third issue.

- The LLVM exception is there for 2 reasons:

  - It removes a potential incompatibility with using LLVM code in combinationwith GPLv2 code.

  - It removes the requirement for developers using LLVM tools to tell the usersof the binaries they produce that those binaries may contain some codeoriginating from LLVM. Such a situation can easily arise when parts of theLLVM run-time libraries are linked in as part of the normal compilationprocess.

    The LLVM exception enables the run-time libraries to be covered by the exactsame single license as the rest of the code base.

# Getting all code covered by the new license

After a decision was made of what the new license should be, we started workingon having all code covered by it.

As a first step, we made sure that all new contributions were covered by the newlicense. This happened after the 8.0 release branch was created. The 100kcommits since are covered by the new license.

The remaining task is to also get the earlier contributions covered by the newlicense. This consists of about 300k commits totaling about 32 million lines ofcontributed code.

![Getting all code covered](https://blog.llvm.org/img/relicensing2021/getting_all_code_covered.png)

What needs to be done to get those earlier contributions covered?

The reason we need a license in the first place is copyright. Most codecontributions are covered by copyright. Which means that the person or companyowning the copyright has a lot of decision power over what can legally be donewith that code. By covering the code with a license, it becomes clear whatothers are permitted to do with that code. If there isn’t a license associatedwith copyrighted code there isn’t all that much useful that others can do withit.

Basically, to get existing code to be covered by a new license, we need to findwho owns the copyright on it, and ask them to agree with offering theircopyrighted work under the new license.

The copyright owner can be either an individual, for example the person whowrote the code originally; or a company, for example a company that employed theperson who wrote the code.

# Asking agreement from copyright owners

We started asking for their agreements. By examining the version control log ofthe 300k-ish commits we need to get agreements for, we found that about 2800different people or email addresses made a contribution.

We reached out to all of them and asked them two things. First, if anycorporation may own the copyright on any of their contributions. Secondly, ifthey agree with relicensing the contributions they copyright own personally.

So far, we’ve heard of about 220 unique corporations potentially copyrightowning some past contribution. We also started reaching out to those, but havenot asked every single one just yet.

# Status as of November 2021

So after 2.5 years since we started asking – where are we with gettingagreements to relicense?

The chart below summarize the current status. It shows a treemap where eachrectangle represents the contributions made by one person. The size of eachrectangle represents how many lines of code the person contributed.

![Treemap showing LLVM relicensing status](https://blog.llvm.org/img/relicensing2021/relicensing-status-treemap.svg)

When the rectangle is green, it means all their contributions are fully coveredby relicensing agreements.

When the rectangle is orange, it means we have not yet received such anagreement. When the rectangle is orange with green stars, it means somecontributions by that person are covered and others not. This can happen forexample when the person has worked for multiple companies over time and onlysome of those companies have agreed with the relicensing so far.

We already have over 94% of all contributed lines of code between 2001 and 2019covered by a licensing agreement. We only have a good 5% of lines of code to gostill.

As you can see, most of the missing agreements are with “long tail”contributors. By long tail contributors here we mean the many contributors whocontributed relatively fewer lines of code. We focussed on reaching out to thebigger contributors first. To reach out well to the long tail of contributors,we’re hoping to get help from the wider LLVM community.

# Help wanted!

Please do consider helping us with reaching any of the people or corporations inthe long tail. Please have a look at [the up-to-date list of people andcorporations](https://docs.google.com/spreadsheets/d/18_0Hog_eSwES8lKwf7WJal3yBwwcYfvPu1yCfZnTcek/edit?usp=sharing) we can use help with getting in touch with.

You can find more detailed guidance on how you can help on the [LLVM foundationwebsite’s relicensing long tailpage](https://foundation.llvm.org/docs/relicensing_long_tail/).

If you do think you could help us with reaching out to someone on the list, oryou may have some other information that could help us, please do let us know byemailing [license-questions@llvm.org](mailto:license-questions@llvm.org).

# Relicensing end game

We are currently in the phase of getting as many relicensing agreements aspossible. We do expect that we may not be able to get absolutely 100% of allpast contributions covered by an agreement. What can we do to achieve currenttop-of-mainline to be fully covered by the new license?

We will need to decide on a contribution-by-contribution basis what the optionsavailable are to achieve that goal. We have at least the following options.

- We can check if copyright even applies to the particular contribution. Verysmall contributions may not be covered by copyright, and hence may not need alicense agreement.

- It may well be that code contributed a long time ago is no longer in the codebase.

- If copyright does apply and the code is still in the code base, we can removethe contribution. Depending on whether current contributors and users stillvalue the effect of that contribution, it may need to be reimplemented.
