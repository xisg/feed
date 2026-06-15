---
title: Bypassing Kernel32.dll for Fun and Nonprofit
url: https://ziglang.org/devlog/2026/#2026-02-03
published: "2026-02-03T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-02-03
---

# [Bypassing Kernel32.dll for Fun and Nonprofit](\#2026-02-03)

Author: Andrew Kelley

The Windows operating system provides a large ABI surface area for doing things in the kernel. However, not all ABIs are created equally. As Casey Muratori points out in his lecture, [The Only Unbreakable Law](https://www.youtube.com/watch?v=5IUj1EZwpJY), the organizational structure of software development teams has a direct impact on the structure of the software they produce.

The DLLs on Windows are organized into a heirarchy, with some of the APIs being high-level wrappers around lower-level ones. For example, whenever you call functions of `kernel32.dll`, ultimately, the actual work is done by `ntdll.dll`. You can observe this directly by using ProcMon.exe and examining stack traces.

What we’ve learned empirically is that the ntdll APIs are generally well-engineered, reasonable, and powerful, but the kernel32 wrappers introduce unnecessary heap allocations, additional failure modes, unintentional CPU usage, and bloat.

This is why the Zig standard library policy is to [Prefer the Native API over Win32](https://codeberg.org/ziglang/zig/issues/31131). We’re not quite there yet - we have plenty of calls into kernel32 remaining - but we’ve taken great strides recently. I’ll give you two examples.

## Example 1: Entropy

According to the official documentation, Windows does not have a straightforward way to get random bytes.

[Many projects including Chromium, boringssl, Firefox, and Rust](https://github.com/rust-random/rand/issues/111) call `SystemFunction036` from `advapi32.dll` because it worked on versions older than Windows 8.

Unfortunately, starting with Windows 8, the first time you call this function, it dynamically loads `bcryptprimitives.dll` and calls [ProcessPrng](https://learn.microsoft.com/en-us/windows/win32/seccng/processprng). If loading the DLL fails (for example due to an overloaded system, which we have observed on Zig CI several times), it returns error 38 (from a function that has `void` return type and is documented to never fail).

The first thing `ProcessPrng` does is heap allocate a small, constant number of bytes. If this fails it returns `NO_MEMORY` in a `BOOL` (documented behavior is to never fail, and always return `TRUE`).

`bcryptprimitives.dll` apparently also runs a test suite every time you load it.

All that `ProcessPrng` is _really_ doing is `NtOpenFile` on `"\\Device\\CNG"` and reading 48 bytes with `NtDeviceIoControlFile` to get a seed, and then initializing a per-CPU AES-based CSPRNG.

So the dependency on `bcryptprimitives.dll` and `advapi32.dll` can both be avoided, and the nondeterministic failure and latencies on first RNG read can also be avoided.

## Example 2: NtReadFile and NtWriteFile

`ReadFile` looks like this:

```zig
pub extern "kernel32" fn ReadFile(
    hFile: HANDLE,
    lpBuffer: LPVOID,
    nNumberOfBytesToRead: DWORD,
    lpNumberOfBytesRead: ?*DWORD,
    lpOverlapped: ?*OVERLAPPED,
) callconv(.winapi) BOOL;

```

`NtReadFile` looks like this:

```zig
pub extern "ntdll" fn NtReadFile(
    FileHandle: HANDLE,
    Event: ?HANDLE,
    ApcRoutine: ?*const IO_APC_ROUTINE,
    ApcContext: ?*anyopaque,
    IoStatusBlock: *IO_STATUS_BLOCK,
    Buffer: *anyopaque,
    Length: ULONG,
    ByteOffset: ?*const LARGE_INTEGER,
    Key: ?*const ULONG,
) callconv(.winapi) NTSTATUS;

```

As a reminder, _the above function is implemented by calling the below function_.

Already we can see some nice things about using the lower level API. For instance, the _real_ API simply gives us the error code as the return value, while the kernel32 wrapper hides the status code somewhere, returns a `BOOL` and then requires you to call `GetLastError` to find out what went wrong. Imagine! Returning a value from a function 🌈

Furthermore, `OVERLAPPED` is a fake type. The Windows kernel doesn’t actually know or care about it at all! The actual primitives here are events, APCs, and `IO_STATUS_BLOCK`.

If you have a synchronous file handle, then `Event` and `ApcRoutine` must be `null`. You get the answer in the `IO_STATUS_BLOCK` immediately. If you pass an APC routine here then some old bitrotted 32-bit code runs and you get garbage results.

On the other hand if you have an asynchronous file handle, then you need to either use an `Event` or an `ApcRoutine`. `kernel32.dll` uses events, which means that it’s doing extra, unnecessary resource allocation and management just to read from a file. Instead, Zig now passes an APC routine and then calls `NtDelayExecution`. This integrates seamlessly with cancelation, making it possible to cancel tasks while they perform file I/O, regardless of whether the file was opened in synchronous mode or asynchronous mode.

For a deeper dive into this topic, please refer to this issue:

[Windows: Prefer the Native API over Win32](https://codeberg.org/ziglang/zig/issues/31131)
