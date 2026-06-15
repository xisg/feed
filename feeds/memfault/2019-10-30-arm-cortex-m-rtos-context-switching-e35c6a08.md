---
title: ARM Cortex-M RTOS Context Switching
url: https://interrupt.memfault.com/blog/cortex-m-rtos-context-switching
published: "2019-10-30T00:00:00Z"
feed: memfault
guid: https://interrupt.memfault.com/blog/cortex-m-rtos-context-switching
---

# ARM Cortex-M RTOS Context Switching

In this article we will explore how **context switching** works on ARM Cortex-M MCUs. We will
discuss how the hardware was designed to support this operation, features that impact the context
switching implementation such as the Floating Point Unit (FPU), and common pitfalls seen when
porting an RTOS to a platform. We will also walk through a practical example of analyzing the
**FreeRTOS** context switcher, `xPortPendSVHandler`, utilizing `gdb` to strengthen our understanding.

[**Continue reading…**](https://interrupt.memfault.com/blog/cortex-m-rtos-context-switching)
