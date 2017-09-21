---
layout: post
title:  ps 命令
date:   2017-09-20 01:08:00 +0800
categories: 常用命令
tag: ps 命令
---

* content
{:toc}

android系统查看zygote进程 fork了哪些进程，又拥有哪些线程
先用
```
# ps
USER      PID   PPID  VSIZE  RSS   WCHAN            PC  NAME
root      164   1     1528996 97064 poll_sched af40e634 S zygote
再用
# ps -t | grep 164
root      164   1     1528996 97064 poll_sched af40e634 S zygote
root      971   164   1528996 97064 futex_wait af3dd3e8 S ReferenceQueueD
root      972   164   1528996 97064 futex_wait af3dd3e8 S FinalizerDaemon
root      973   164   1528996 97064 futex_wait af3dd3e8 S FinalizerWatchd
root      974   164   1528996 97064 futex_wait af3dd3e8 S HeapTaskDaemon
system    400   164   1658516 125052 SyS_epoll_ af40e448 S system_server
u0_a19    479   164   1108332 148548 SyS_epoll_ af40e448 S com.android.systemui
radio     569   164   961720 73624 SyS_epoll_ af40e448 S com.android.phone
system    591   164   975488 64968 SyS_epoll_ af40e448 S com.android.settings
u0_a8     738   164   945584 48848 SyS_epoll_ af40e448 S android.ext.services
u0_a31    755   164   953132 59304 SyS_epoll_ af40e448 S com.android.deskclock
u0_a49    770   164   947444 58776 SyS_epoll_ af40e448 S com.android.inputmethod.pinyin
u0_a46    798   164   947024 51816 SyS_epoll_ af40e448 S com.android.printspooler
system    813   164   946604 48712 SyS_epoll_ af40e448 S com.android.keychain
u0_a10    829   164   983404 91752 SyS_epoll_ af40e448 S com.android.launcher
u0_a7     858   164   952688 65064 SyS_epoll_ af40e448 S android.process.media
u0_a26    888   164   952460 57636 SyS_epoll_ af40e448 S com.android.calendar
u0_a2     911   164   950316 56620 SyS_epoll_ af40e448 S com.android.providers.calendar
u0_a33    924   164   960964 63680 SyS_epoll_ af40e448 S com.android.email
u0_a11    954   164   947080 48996 SyS_epoll_ af40e448 S com.android.managedprovisioning
u0_a13    970   164   945428 48432 SyS_epoll_ af40e448 S com.android.onetimeinitializer
```
其中PPID 为164的进程都是Zygote fork的，  VSZIE与zygote相同的是zygote的子线程， 其余是子进程。

**线程与进程的最为本质的区别便是是否共享内存空间，图中VSIZE和Zygote进程相同的才是Zygote的子线程，
否则就是Zygote的子进程**
