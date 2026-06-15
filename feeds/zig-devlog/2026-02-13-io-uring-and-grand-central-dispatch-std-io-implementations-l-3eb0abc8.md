---
title: io_uring and Grand Central Dispatch std.Io implementations landed
url: https://ziglang.org/devlog/2026/#2026-02-13
published: "2026-02-13T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-02-13
---

# io_uring and Grand Central Dispatch std.Io implementations landed

Author: Andrew Kelley

As we approach the end of the 0.16.0 release cycle, Jacob has been hard at work, bringing `std.Io.Evented` up to speed with all the latest API changes:

- [io\_uring implementation](https://codeberg.org/ziglang/zig/pulls/31158)
- [Grand Central Dispatch implementation](https://codeberg.org/ziglang/zig/pulls/31198)

Both of these are based on userspace stack switching, sometimes called “fibers”, “stackful coroutines”, or “green threads”.

They are now **available to tinker with**, by constructing one’s application using `std.Io.Evented`. They should be considered **experimental** because there is important followup work to be done before they can be used reliably and robustly:

- [better error handling](https://codeberg.org/ziglang/zig/issues/31199)
- remove the logging
- diagnose the unexpected performance degradation when using `IoMode.evented` for the compiler
- [a couple functions still unimplemented](https://codeberg.org/ziglang/zig/issues/31200)
- more test coverage is needed
- [builtin function to tell you the maximum stack size of a given function](https://github.com/ziglang/zig/issues/157) to make these implementations practical to use when overcommit is off.

With those caveats in mind, it seems we are indeed reaching the Promised Land, where Zig code can have Io implementations effortlessly swapped out:

```zig
const std = @import("std");

pub fn main(init: std.process.Init.Minimal) !void {
    var debug_allocator: std.heap.DebugAllocator(.{}) = .init;
    const gpa = debug_allocator.allocator();

    var threaded: std.Io.Threaded = .init(gpa, .{
        .argv0 = .init(init.args),
        .environ = init.environ,
    });
    defer threaded.deinit();
    const io = threaded.io();

    return app(io);
}

fn app(io: std.Io) !void {
    try std.Io.File.stdout().writeStreamingAll(io, "Hello, World!\n");
}

```

```
$ strace ./hello_threaded
execve("./hello_threaded", ["./hello_threaded"], 0x7ffc1da88b20 /* 98 vars */) = 0
mmap(NULL, 262207, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f583f338000
arch_prctl(ARCH_SET_FS, 0x7f583f378018) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
prlimit64(0, RLIMIT_STACK, {rlim_cur=16384*1024, rlim_max=RLIM64_INFINITY}, NULL) = 0
sigaltstack({ss_sp=0x7f583f338000, ss_flags=0, ss_size=262144}, NULL) = 0
sched_getaffinity(0, 128, [0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31]) = 8
rt_sigaction(SIGIO, {sa_handler=0x1019d90, sa_mask=[], sa_flags=SA_RESTORER, sa_restorer=0x10328c0}, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGPIPE, {sa_handler=0x1019d90, sa_mask=[], sa_flags=SA_RESTORER, sa_restorer=0x10328c0}, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
writev(1, [{iov_base="Hello, World!\n", iov_len=14}], 1Hello, World!
) = 14
rt_sigaction(SIGIO, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=SA_RESTORER, sa_restorer=0x10328c0}, NULL, 8) = 0
rt_sigaction(SIGPIPE, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=SA_RESTORER, sa_restorer=0x10328c0}, NULL, 8) = 0
exit_group(0)                           = ?
+++ exited with 0 +++

```

Swapping out only the I/O implementation:

```zig
const std = @import("std");

pub fn main(init: std.process.Init.Minimal) !void {
    var debug_allocator: std.heap.DebugAllocator(.{}) = .init;
    const gpa = debug_allocator.allocator();

    var evented: std.Io.Evented = undefined;
    try evented.init(gpa, .{
        .argv0 = .init(init.args),
        .environ = init.environ,
        .backing_allocator_needs_mutex = false,
    });
    defer evented.deinit();
    const io = evented.io();

    return app(io);
}

fn app(io: std.Io) !void {
    try std.Io.File.stdout().writeStreamingAll(io, "Hello, World!\n");
}

```

```
execve("./hello_evented", ["./hello_evented"], 0x7fff368894f0 /* 98 vars */) = 0
mmap(NULL, 262215, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f70a4c28000
arch_prctl(ARCH_SET_FS, 0x7f70a4c68020) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
prlimit64(0, RLIMIT_STACK, {rlim_cur=16384*1024, rlim_max=RLIM64_INFINITY}, NULL) = 0
sigaltstack({ss_sp=0x7f70a4c28008, ss_flags=0, ss_size=262144}, NULL) = 0
sched_getaffinity(0, 128, [0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31]) = 8
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f70a4c27000
mmap(0x7f70a4c28000, 548864, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f70a4ba1000
io_uring_setup(64, {flags=IORING_SETUP_COOP_TASKRUN|IORING_SETUP_SINGLE_ISSUER, sq_thread_cpu=0, sq_thread_idle=1000, sq_entries=64, cq_entries=128, features=IORING_FEAT_SINGLE_MMAP|IORING_FEAT_NODROP|IORING_FEAT_SUBMIT_STABLE|IORING_FEAT_RW_CUR_POS|IORING_FEAT_CUR_PERSONALITY|IORING_FEAT_FAST_POLL|IORING_FEAT_POLL_32BITS|IORING_FEAT_SQPOLL_NONFIXED|IORING_FEAT_EXT_ARG|IORING_FEAT_NATIVE_WORKERS|IORING_FEAT_RSRC_TAGS|IORING_FEAT_CQE_SKIP|IORING_FEAT_LINKED_FILE|IORING_FEAT_REG_REG_RING|IORING_FEAT_RECVSEND_BUNDLE|IORING_FEAT_MIN_TIMEOUT|IORING_FEAT_RW_ATTR|IORING_FEAT_NO_IOWAIT, sq_off={head=0, tail=4, ring_mask=16, ring_entries=24, flags=36, dropped=32, array=2112, user_addr=0}, cq_off={head=8, tail=12, ring_mask=20, ring_entries=28, overflow=44, cqes=64, flags=40, user_addr=0}}) = 3
mmap(NULL, 2368, PROT_READ|PROT_WRITE, MAP_SHARED|MAP_POPULATE, 3, 0) = 0x7f70a4ba0000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_SHARED|MAP_POPULATE, 3, 0x10000000) = 0x7f70a4b9f000
io_uring_enter(3, 1, 1, IORING_ENTER_GETEVENTS, NULL, 8Hello, World!
) = 1
io_uring_enter(3, 1, 1, IORING_ENTER_GETEVENTS, NULL, 8) = 1
munmap(0x7f70a4b9f000, 4096)            = 0
munmap(0x7f70a4ba0000, 2368)            = 0
close(3)                                = 0
munmap(0x7f70a4ba1000, 548864)          = 0
exit_group(0)                           = ?
+++ exited with 0 +++

```

Key point here being that the `app` function is identical between those two snippets.

Moving beyond Hello World, the Zig compiler itself works fine using `std.Io.Evented`, both with io\_uring and with GCD, but as mentioned above, there is a not-yet-diagnosed performance degradation when doing so.

Happy hacking,

Andrew
