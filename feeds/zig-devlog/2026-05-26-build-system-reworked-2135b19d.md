---
title: Build System Reworked
url: https://ziglang.org/devlog/2026/#2026-05-26
published: "2026-05-26T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-05-26
---

# [Build System Reworked](\#2026-05-26)

Author: Andrew Kelley

Big branch just landed: [separate the maker process from the configurer process](https://codeberg.org/ziglang/zig/pulls/35428)

This devlog entry is essentially a preview of the upcoming release notes, but serves as an advanced notice to those who want to help test out the new features and provide feedback that will guide the Zig project moving forward.

Before, `build.zig` files plus the build system implementation were all compiled into one bloated process, in Debug mode. After `build.zig` logic finished constructing a build graph in memory, the “build runner” code executed it.

Now, `build.zig` files are compiled into a small process (the “configurer”) in debug mode. After this logic finishes constructing a build graph in memory, it is serialized to a binary configuration file. The parent `zig build` process is aware of this file and caches it for next time. While waiting for all that, it asynchronously compiles the build graph execution process (the “maker”) in release mode. Once the configuration file is available and the maker process is finished compiling, the maker process is executed, passing it the configuration file. The maker process only needs to be compiled once per `zig version` thanks to the global cache. The maker process then executes the build graph, which is contained within the serialized configuration file.

The primary motivation of this change was to make `zig build` faster, in three ways:

1. Only the user’s `build.zig` logic will be compiled with each change, rather than the entire build system along with it. This is starting to become more valuable now that we have introduced `--watch`, `--fuzz` and `--webui`. The build system can grow more features without making `zig build` take longer.

2. Now the build system can skip rerunning the `build.zig` logic entirely when it knows nothing will change, for example if you add `-freference-trace` to your `zig build` command line, it now avoids re-running your `build.zig` logic redundantly, using the same configuration as last time.

3. Now the process that actually executes the build graph is compiled with optimizations enabled.


To demonstrate points 2 and 3, here is the difference between running `zig build --help` before and after:

```
Benchmark 1 (34 runs): master/zig build -h
  measurement          mean ± σ            min … max           outliers         delta
  wall_time           150ms ± 5.52ms     145ms …  165ms          4 (12%)        0%
  peak_rss           84.8MB ±  275KB    84.2MB … 85.1MB          0 ( 0%)        0%
  cpu_cycles          593M  ± 4.01M      588M  …  608M           2 ( 6%)        0%
  instructions        995M  ± 52.5K      995M  …  995M           0 ( 0%)        0%
  cache_references   25.8M  ±  165K     25.4M  … 26.1M           0 ( 0%)        0%
  cache_misses        651K  ± 20.1K      619K  …  697K           0 ( 0%)        0%
  branch_misses       918K  ± 7.44K      906K  …  935K           0 ( 0%)        0%
Benchmark 2 (348 runs): branch/zig build -h
  measurement          mean ± σ            min … max           outliers         delta
  wall_time          14.3ms ±  744us    13.2ms … 23.3ms          8 ( 2%)        ⚡- 90.4% ±  0.4%
  peak_rss           78.5MB ±  562KB    77.1MB … 81.4MB          7 ( 2%)        ⚡-  7.4% ±  0.2%
  cpu_cycles         24.1M  ±  821K     22.8M  … 27.1M           3 ( 1%)        ⚡- 95.9% ±  0.1%
  instructions       43.7M  ± 23.8K     43.7M  … 43.8M          56 (16%)        ⚡- 95.6% ±  0.0%
  cache_references   1.46M  ± 14.6K     1.40M  … 1.50M          19 ( 5%)        ⚡- 94.3% ±  0.1%
  cache_misses        142K  ± 4.87K      127K  …  157K           2 ( 1%)        ⚡- 78.1% ±  0.4%
  branch_misses       126K  ± 1.37K      120K  …  129K          12 ( 3%)        ⚡- 86.3% ±  0.1%

```

It’s dramatic because before, `build.zig` logic was being executed with each `zig build` command, but now, the build system uses the cached, serialized configuration instead.

Aside from performance, I expect third-party tooling such as ZLS to benefit from consuming the serialized configuration file rather than maintaining a fork of the build runner.

This changeset heavily reworks the internal mechanism of the zig build system, however, it is mostly non-breaking from an API perspective, with the exceptions noted in the PR linked above.

For most people I’m guessing this is the main breaking change they’ll hit:

```zig
if (b.args) |args| {
    run_cmd.addArgs(args);
}

```

⬇️

```zig
run_cmd.addPassthruArgs();

```

This removes a capability from build scripts since they can no longer observe those arguments. In exchange, it means that when changing those arguments, build scripts no longer must be rebuilt from source.

If you’re someone who wants to influence the direction of Zig, this is a good time to upgrade your projects to the development version and try out these changes. We’ll be releasing 0.17.0 within a couple weeks from now. However, if you don’t have time, and you find out that 0.17.0 broke your build, don’t worry, there will be plenty of opportunity to get fixes in for the 0.17.1 tag as well.
