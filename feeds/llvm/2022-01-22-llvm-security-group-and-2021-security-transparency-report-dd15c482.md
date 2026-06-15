---
title: LLVM security group and 2021 security transparency report
url: https://blog.llvm.org/posts/2022-01-22-security-group-transparency-reports/
published: "2022-01-22T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2022-01-22-security-group-transparency-reports/
---

# LLVM security group and 2021 security transparency report

Over the past few years, the LLVM project has seen the creation of a securitygroup, which aims to enable responsible disclosure and fixing ofsecurity-related issues affecting the LLVM project.

The [LLVM security group](https://llvm.org/docs/Security.html) was establishedon the 10th of July 2020 by the act of the [initialcommit](https://github.com/llvm/llvm-project/commit/7bf73bcf6d93) describingthe purpose of the group and the processes it follows. Many of the group’sprocesses were still not well-defined enough for the group to operate well.Over the course of 2021, the key processes were defined well enough to enablethe group to operate reasonably well:

- We defined details on how to report security issues, see [this commit on20th of May 2021](https://github.com/llvm/llvm-project/commit/c9dbaa4c86d2).
- We refined the nomination process for new group members, see [thiscommit on 30th of July2021](https://github.com/llvm/llvm-project/commit/4c98e9455aad).
- We wrote a first annual transparency report, which is published at [https://llvm.org/docs/SecurityTransparencyReports.html](https://llvm.org/docs/SecurityTransparencyReports.html) There is a copy of the 2021 transparency report below.

Over the course of 2021, we had 2 people leave the LLVM Security group and 4people join.

In 2021, the security group received 13 issue reports that were made publiclyvisible before 31st of December 2021. The security group judged 2 of thesereports to be security issues:

- [https://bugs.chromium.org/p/llvm/issues/detail?id=5](https://bugs.chromium.org/p/llvm/issues/detail?id=5)
- [https://bugs.chromium.org/p/llvm/issues/detail?id=11](https://bugs.chromium.org/p/llvm/issues/detail?id=11)

Both issues were addressed with source changes: #5 in clangd/vscode-clangd, and#11 in llvm-project. No dedicated LLVM release was made for either.

We believe that with the publishing of the first annual transparency report,the security group now has implemented all necessary processes for the group tooperate as promised. The group’s processes can be improved further, and we doexpect further improvements to get implemented in 2022. Many of the potentialimprovements end up being discussed on the [monthly public call on LLVM’ssecurity group](https://llvm.org/docs/GettingInvolved.html#online-sync-ups).
