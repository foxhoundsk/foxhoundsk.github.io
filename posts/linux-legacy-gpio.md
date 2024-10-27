---
title: "So you want to use legacy Linux GPIO?"
date: 2024-09-06T00:32:06+08:00
draft: false
description: Linux GPIO
isStarred: false
---

Starting kernel 4.8, a new GPIO interface, GPIO character device, replaced the legacy GPIO interface. The cons of the legacy interface are:

- State not tied to process
- Concurrent access to sysfs attributes
- If process crashes, the GPIOs remain exported
- Cumbersome API
- Single sequence of GPIO numbers representing a two-level hierarchy - necessary to calculate the number of the GPIO, numbers not stable

For details, see the [maintainer's slides](https://ostconf.com/system/attachments/files/000/001/532/original/Linux_Piter_2018_-_New_GPIO_interface_for_linux_userspace.pdf).

However, there is still plenty of old kernel users relying on the legacy GPIO.

The GPIO event consuming, however, typically requires 2 syscalls to do the trick, which are read(2) and lseek(2). Alternatively, one can also close-and-open after the read, which requires 3 syscalls.

Actually, it can be done with a single syscall, which is pread(2).

pread(2) has the following signature:

{% highlight c %}
ssize_t pread(int fd, void buf[.count], size_t count, off_t offset);
{% endhighlight %}

As the last parameter (i.e. `offset`) suggests, using pread(2) would not update the file offset after the read. Meaning one would not need to reset the file offset manually.

So, it is recommended to use pread(2) for consuming the event.

You would not like additional syscalls to harm your CPU cache and the like. Besides, issuing non-vdso syscalls is never a cheap act.

---

This post is here because there would probably no one interested in checking the kernel document, which I [recently added](https://github.com/torvalds/linux/commit/7f1e45f4ae7671550e15354ef87194bccd99ecec) the above recommendadtion.
