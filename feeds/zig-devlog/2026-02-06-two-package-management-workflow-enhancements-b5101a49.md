---
title: Two Package Management Workflow Enhancements
url: https://ziglang.org/devlog/2026/#2026-02-06
published: "2026-02-06T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-02-06
---

# [Two Package Management Workflow Enhancements](\#2026-02-06)

Author: Andrew Kelley

If you have a Zig project with dependencies, two big changes just landed which I think you will be interested to learn about.

Fetched packages are now stored _locally_ in the `zig-pkg` directory of the project root (next to your `build.zig` file).

For example here are a few results from [awebo](https://codeberg.org/awebo-chat/awebo) after running `zig build`:

```
$ du -sh zig-pkg/*
13M    freetype-2.14.1-alzUkTyBqgBwke4Jsot997WYSpl207Ij9oO-2QOvGrOi
20K    opus-0.0.2-vuF-cMAkAADVsm707MYCtPmqmRs0gzg84Sz0qGbb5E3w
4.3M   pulseaudio-16.1.1-9-mk_62MZkNwBaFwiZ7ZVrYRIf_3dTqqJR5PbMRCJzSuLw
5.2M   uucode-0.1.0-ZZjBPvtWUACf5dqD_f9I37VGFsN24436CuceC5pTJ25n
728K   vaxis-0.5.1-BWNV_AxECQCj3p4Hcv4U3Yo1WMUJ7Z2FUj0UkpuJGxQQ

```

It is highly recommended to add this directory to the project-local source control ignore file (e.g. `.gitignore`). However, by being outside of `.zig-cache`, it provides the possibility of distributing self-contained source tarballs, which contain all dependencies and therefore can be used to build offline, or for archival purposes.

Meanwhile, an _additional_ copy of the dependency is cached globally. After filtering out all the unused files based on the `paths` filter, the contents are recompressed:

```
$ du -sh ~/.cache/zig/p/*
2.4M    freetype-2.14.1-alzUkTyBqgBwke4Jsot997WYSpl207Ij9oO-2QOvGrOi.tar.gz
4.0K    opus-0.0.2-vuF-cMAkAADVsm707MYCtPmqmRs0gzg84Sz0qGbb5E3w.tar.gz
636K    pulseaudio-16.1.1-9-mk_62MZkNwBaFwiZ7ZVrYRIf_3dTqqJR5PbMRCJzSuLw.tar.gz
880K    uucode-0.1.0-ZZjBPvtWUACf5dqD_f9I37VGFsN24436CuceC5pTJ25n.tar.gz
120K    vaxis-0.5.1-BWNV_BFECQBbXeTeFd48uTJRjD5a-KD6kPuKanzzVB01.tar.gz

```

The motivation for this change is to make it easier to tinker. Go ahead and edit those files, see what happens. Swap out your package directory with a git clone. Grep your dependencies all together. Configure your IDE to auto-complete based on the `zig-pkg` directory. [Run baobab on your dependency tree](https://codeberg.org/awebo-chat/awebo/issues/61). Furthermore, by having the global cache have compressed files instead makes it easier to share that cached data between computers. In the future, [it is planned to support peer-to-peer torrenting of dependency trees](https://github.com/ziglang/zig/issues/23236). By recompressing packages into a canonical form, this will allow peers to share Zig packages with minimal bandwidth. I love this idea because it simultaneously provides resilience to network outages, as well as a popularity contest. Find out which open source packages are popular based on number of seeders!

The second change here is the addition of the `--fork` flag to `zig build`.

In retrospect, it seems so obvious, I don’t know why I didn’t think of it since the beginning. It looks like this:

```
zig build --fork=[path]

```

This is a **project override** option. Given a path to a source checkout of a project, all packages matching that project across the entire dependency tree will be overridden.

Thanks to the fact that package content hashes include name and fingerprint, **this resolves before the package is potentially fetched**.

This is an easy way to temporarily use one or more forks which are in entirely separate directories. You can iterate on your entire dependency tree until everything is working, while using comfortably the development environment and source control of the dependency projects.

The fact that it is a CLI flag makes it appropriately ephemeral. The moment you drop the flags, you’re back to using your pristine, fetched dependency tree.

If the project does not match, an error occurs, preventing confusion:

```
$ zig build --fork=/home/andy/dev/mime
error: fork /home/andy/dev/mime matched no mime packages
$

```

If the project does match, you get a reminder that you are using a fork, preventing confusion:

```
$ zig build --fork=/home/andy/dev/dvui
info: fork /home/andy/dev/dvui matched 1 (dvui) packages
...

```

This functionality is intended to enhance the workflow of dealing with ecosystem breakage. I already tried it a bit and found it to be quite pleasant to work with. The new workflow goes like this:

1. Fail to build from source due to ecosystem breakage.
2. Tinker with `--fork` until your project works again. During this time you can use the actual upstream source control, test suite, `zig build test --watch -fincremental`, etc.
3. Now you have a new option: be selfish and just keep working on your own stuff, or you can proceed to submit your patches upstream.

…and you can probably skip the step where you switch your `build.zig.zon` to your fork unless you expect upstream to take a long time to merge your fixes.
