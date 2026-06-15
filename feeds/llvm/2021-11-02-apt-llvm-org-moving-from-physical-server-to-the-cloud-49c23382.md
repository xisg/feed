---
title: apt.llvm.org - moving from physical server to the cloud
url: https://blog.llvm.org/posts/2021-11-02-apt.llvm.org-moving-from-physical-server-to-the-cloud/
published: "2021-11-02T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2021-11-02-apt.llvm.org-moving-from-physical-server-to-the-cloud/
---

# apt.llvm.org - moving from physical server to the cloud

In this blog post, I would like to explain how I migrated [apt.llvm.org](https://apt.llvm.org) from physical hardware hosted ina datacenter to the Google cloud. This has resulted in better securityand faster builds for LLVM Debian/Ubuntu nightly builds.

# Previous infrastructure

About 10 years ago, Nick Lewycky from Google offered to replace my mixof old and terrible servers with 16 blade servers in a chassis and acontrol server. Below are pictures of the front and back of theseservers, which have served us faithfully for 10 years:

* * *

![Old server - front](https://blog.llvm.org/img/gcp/servers-front.png)![Old server - back](https://blog.llvm.org/img/gcp/servers-back.png)

* * *

The hosting was sponsored by [_IRILL_](https://www.irill.org/), atInria, next to Versailles.

This setup was running using a PXE install with preseed to set upeverything on the 16 blades.

While it served us very well for years, it is now holding us back.

- Some of the blades are dying;
- Old hardware - it takes from 4 to 7 hours per build;
- All blades are always on even if no build is executed;
- Even if it happened rarely, I had to drive to the DC to fix somesystems.

# New infrastructure

As part of the [_OpenSSF initiative_](https://openssf.org/), Google andthe Linux foundation offered a budget to migrate this service on GoogleCloud Platform (GCP).

As Jenkins has been working very well, I reused this approach andimported the previous configurations (available on [_https://github.com/opencollab/llvm-jenkins.debian.net/tree/master/jobs_](https://github.com/opencollab/llvm-jenkins.debian.net/tree/master/jobs)).

I followed most of the tutorial provided on GCP, [_Using Jenkins fordistributed builds on ComputeEngine_](https://cloud.google.com/architecture/using-jenkins-for-distributed-builds-on-compute-engine).

The architecture is the following:

![The schema of the infra](https://blog.llvm.org/img/gcp/gcp-archi-infra.png)

Similar to puppet or chef, Salt is the configuration management toolused to configure the various systems. A salt server configures both theJenkins controller and the reference node. This node is a preconfiguredVM with all the dependencies and tools to start the build. This node ishalted most of the time and only started when the image needs to beupdated.

For example, the list of Virtual Machines now usually looks like thefollowing screenshot:

![GCP - list of servers](https://blog.llvm.org/img/gcp/gcp-list-server.png)

- 5 build VMs - off after 5 minutes without activity
- The template VM (off as it is only started to update the image)
- The jenkins orchestrator

## Build servers

The controller is always on and starts/ends the build nodes.

The reference node is configured and updated using this script, based ongcloud, the Google cloud CLI:

[_https://github.com/opencollab/llvm-jenkins.debian.net/blob/master/image-update.sh_](https://github.com/opencollab/llvm-jenkins.debian.net/blob/master/image-update.sh)

This script performs the following steps:

- starts the reference node and then ssh into it.
- Perform the needed changes (ex: new chroot for a new distro version,new build hooks, etc).
- Once the changes are performed.

Usually:

- Salt update
- Refresh the chroots
- _git pull_ llvm-project
- Stops the VM
- Creates a new image.
- Archives the old image.
- This image will be used by Jenkins to start new VMs.

The node image contains the various i386 and amd64 chroot for supportedDebian and Ubuntu installs. The LLVM repository is also checked out toavoid doing a long clone with many files on every VM startup.

Thanks to this, the jobs just need to perform a small _git pull_ insteadof a full clone.

![GCP - image node](https://blog.llvm.org/img/gcp/gcp-image-node.png)

The size of this image is 6.67gb. This approach saves about 10 minutesof the build pipeline (skipping the creation of the chroot + fullclone).

This wasn’t easy. I had to iterate 25 times to have a proper imagesupporting all the use cases.

GCP provides a shared file system mechanism available as an NFS mountagecalled [_Filestore_](https://cloud.google.com/filestore). Each buildnode has access to it and will update the various repositories just likeif it was a local file system.

## Benefits and cost

The first benefit is security. This is based on the Google cloudinfrastructure, so it benefits from all their security work.

The fact that the VMs are always cycled and recreated from a clean imagewill also help. They are not directly accessible from the Internet (theyleverage cloud NAT to be able to download packages from the Internet),so it will be harder to attack.

An amd64 build now runs in parallel takes \*\*about 50 minutes \*\*insteadof almost 7 hours.

In terms of cost, depending on the activity, the daily cost varies from70 to 150 US$/day.

Now that the migration has been completed, it is now possible toconsider some builds leveraging PGO and LTO for faster binaries.

## Thanks

Thanks to Laurent Vaylet from the GCP team for the help and patienceanswering my newbie questions, and also thanks to Google and the LinuxFoundation for the GCP credits and support.
