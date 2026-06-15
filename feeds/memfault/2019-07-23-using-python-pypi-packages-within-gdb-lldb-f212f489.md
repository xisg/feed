---
title: Using Python PyPi Packages within GDB/LLDB
url: https://interrupt.memfault.com/blog/using-pypi-packages-with-GDB
published: "2019-07-23T00:00:00Z"
feed: memfault
guid: https://interrupt.memfault.com/blog/using-pypi-packages-with-GDB
---

# Using Python PyPi Packages within GDB/LLDB

[In a previous\
post](/blog/automate-debugging-with-gdb-python-api), we
discussed how to automate some of the more tedious parts of debugging firmware
using
[Python in GDB Scripts](https://sourceware.org/gdb/onlinedocs/gdb/Python-API.html).
To make these commands more powerful, one could use third-party packages from
Python’s [PyPi](https://pypi.org/) repository. In this post, we will discuss how
to properly setup GDB, Python, and optionally virtualenv and then modify the
`uuid_list_dump` command from the post mentioned above to make use of a third
party package installed through PyPi.

[**Continue reading…**](https://interrupt.memfault.com/blog/using-pypi-packages-with-GDB)
