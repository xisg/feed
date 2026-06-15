---
title: Some news about apt.llvm.org
url: https://blog.llvm.org/2017/03/some-news-about-aptllvmorg.html
published: "2017-03-06T12:59:00Z"
feed: llvm
guid: https://blog.llvm.org/2017/03/some-news-about-aptllvmorg.html
---

# Some news about apt.llvm.org

[apt.llvm.org](http://apt.llvm.org/) provides Debian and Ubuntu repositories for every maintained version of these distributions. LLVM, Clang, clang extra tools, compiler-rt, polly, LLDB and LLD packages are generated for the stable, stabilization and development branches.

As it seems that we have more and more users of these packages, I would like to share an update about various recent changes.

### New features

#### LLD  First, the cool new stuff : lld is now proposed and built for i386/amd64 on all Debian and Ubuntu supported versions. The test suite is also executed and the [coverage results](http://llvm.org/reports/coverage/) are great.

#### 4.0  Then, following the branching for the 4.0 release, I created new repositories to propose this release.  For example, for Debian stable, just add the following in _/etc/apt/sources.list.d/llvm.list_

> _deb http://apt.llvm.org/jessie/ llvm-toolchain-jessie-4.0 main__deb-src http://apt.llvm.org/jessie/ llvm-toolchain-jessie main_

#### llvm-defaults  Obviously, the trunk is now 5.0. If llvm-defaults is used, clang, lldb and other meta packages will be automatically updated to this version.  As a consequence and also because the branches are dead, 3.7 and 3.8 jobs have been disabled. Please note that both repositories are still available on apt.llvm.org and won't be removed.

#### Zesty: New Ubuntu  Packages for the next Ubuntu 17.04 (zesty) are also generated for 3.9, 4.0 and 5.0.

#### libfuzzer  It has been implemented a few months ago but not clearly communicated. [libfuzzer](http://llvm.org/docs/LibFuzzer.html) has also its own packages: libfuzzer-X.Y-dev (example: _libfuzzer-3.9-dev,_ _libfuzzer-4.0-dev_ or _libfuzzer-5.0-dev_).

### Changes in the infrastructure

In order to support the load, I started to use new blades that Google (thanks again to Nick Lewycky) sponsored for an initiative that I was running for Debian and [IRILL](https://www.irill.org/). The 6 new blades removed all the wait time. With a new salt configuration, I automated the deployment of the slaves. In case the load increases again, we will have access to more blades.

I also took the time to fix some long ongoing issues:

- all repositories are signed and verified that they are
- _i386_ and _amd64_ packages are now uploaded at once instead of being uploaded separately. This was causing checksum error when one of the two architectures built correctly and the second was failing (ex: test failing)

Last but not least, the [code coverage results](http://llvm.org/reports/coverage/) are produced in a more reliable manner.

### More information about the implementation and services.

As what is shipped on apt.llvm.org is exactly the same as in Debian and Ubuntu, packaging files are stored on the [Debian subversion server.](https://anonscm.debian.org/viewvc/pkg-llvm/llvm-toolchain/branches/)

A [Jenkins instance](http://llvm-jenkins.debian.net/) is in charge of the orchestration of the whole build infrastructure.

The trunk packages are built twice a day for every Debian and Ubuntu packages. Branches (3.9 and 4.0 currently) are rebuilt only when the - trigger job found a change.

In both case, the Jenkins source job will checkout the Debian SVN branches for their version, checkout/update LLVM/clang/etc repositories and repack everything to create the source tarballs and Debian files (dsc, etc).The completion of job will trigger the binaries job to start. These jobs, thanks to [Debian Jenkins glue](https://jenkins-debian-glue.org/) will create or update Debian/Ubuntu versions.

Then builds are done the usual way through pbuilder for both i386 and amd64. All the test suites are going to be executed. If any LLVM test is failing on i386 or amd64, the whole build will fail. If both builds and the LLVM testsuite are successful, the sync job will start and rsync packages to the LLVM server to be replicated on the CDN. If one or both builds fail, a notification is sent to the administrator.

Some Debian static analysis ( [lintian](https://packages.debian.org/stretch/lintian)) are executed on the packages to prevent some packaging errors. From time to time, some interesting issues are found.

In parallel, some binary builds have some special hooks like [Coverity](https://scan.coverity.com/projects/llvm), [code coverage](http://llvm.org/reports/coverage/) or installation of more recent versions of gcc for Ubuntu precise.

### Report bugs

Bugs can be reported on the [bugzilla of the LLVM project](https://bugs.llvm.org//enter_bug.cgi?product=Packaging&component=deb%20packages) in the product "Packaging" and the component "deb packages".

### Common issues

Because packaging quickly moving projects like LLVM or clang, in some cases, this can be challenging to follow the rhythm in particular with regard to tests. For Debian unstable or the latest version of Ubuntu, the matrix is complexified by new versions of the basic pieces of the operating system like gcc/g++ or libtstdc++.

This is also not uncommon that some tests are being ignored in the process.

### How to help

Some [new comers bugs](https://bugs.debian.org/cgi-bin/pkgreport.cgi?include=originator%3Asylvestre;tag=newcomer) are available. As an example:

- [Move the compiler-rt libraries](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=751789) into a specific packages to simplify their usage by other packages
- [Ship libc++](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=813673) as part of llvm-toolchain packages
- [Ship openmp library](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=813672) as part of llvm-toolchain packages
- [Full bootstrap of llvm-toolchain](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=827185) (use a clang built during the process)
- ...

Related to all this, a Google Summer of Code 2017 under the LLVM umbrella has been proposed: [Integrate libc++ and OpenMP in apt.llvm.org](http://llvm.org/OpenProjects.html#apt)

Help is also needed to keep track of the new test failures and get them fixed upstream. For example, a few tests have been marked as _expected to fail_ to avoid crashes.
