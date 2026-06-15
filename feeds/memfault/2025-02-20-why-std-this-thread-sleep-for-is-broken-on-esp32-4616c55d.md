---
title: Why std::this_thread::sleep_for() is broken on ESP32
url: https://interrupt.memfault.com/blog/why-sleep-for-is-broken-on-esp32
published: "2025-02-20T00:00:00Z"
feed: memfault
guid: https://interrupt.memfault.com/blog/why-sleep-for-is-broken-on-esp32
---

# Why std::this_thread::sleep_for() is broken on ESP32

A curious bug appearing after upgrading to IDF v5 led me into a deep dive of how
`std::this_thread::sleep_for()` is implemented on the ESP32. I discuss how the
IDF implements `pthreads` and `newlib` to provide C++ threading functionality.
The results are surprising: a simple 10 millisecond sleep was killing
performance, but only in the new version of IDF due to an interaction between
`libstdc++` and `usleep()`.

[**Continue reading…**](https://interrupt.memfault.com/blog/why-sleep-for-is-broken-on-esp32)
