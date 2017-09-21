---
layout: post
title:  常用编译器命令
date:   2017-09-06 01:08:00 +0800
categories: 常用命令
tag: 常用编译器命令
---

* content
{:toc}


查找一个binary依赖的库方法：
arm-none-linux-gnueabi-readelf -a fastboot | grep library

通过arm-none-linux-gnueabi-addr2line 同居判断crash日志的地址错误的代码行数

```
01-01 08:00:22.273   155   155 F libc    : stack corruption detected
01-01 08:00:22.273   155   155 F libc    : Fatal signal 6 (SIGABRT), code -6 in tid 155 (surfaceflinger)
01-01 08:00:22.273   142   142 W         : debuggerd: handling request: pid=155 uid=1000 gid=1003 tid=155
01-01 08:00:22.330  1012  1012 F DEBUG   : *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
01-01 08:00:22.330  1012  1012 F DEBUG   : Build fingerprint: 'Android/zx2000/zx2000:7.0/NRD90M/kimihu08151932:userdebug/dev-keys'
01-01 08:00:22.330  1012  1012 F DEBUG   : Revision: '0'
01-01 08:00:22.330  1012  1012 F DEBUG   : ABI: 'arm'
01-01 08:00:22.330  1012  1012 F DEBUG   : pid: 155, tid: 155, name: surfaceflinger  >>> /system/bin/surfaceflinger <<<
01-01 08:00:22.330  1012  1012 F DEBUG   : signal 6 (SIGABRT), code -6 (SI_TKILL), fault addr --------
01-01 08:00:22.332  1012  1012 F DEBUG   : Abort message: 'stack corruption detected'
01-01 08:00:22.332  1012  1012 F DEBUG   :     r0 00000000  r1 0000009b  r2 00000006  r3 00000008
01-01 08:00:22.332  1012  1012 F DEBUG   :     r4 b081c58c  r5 00000006  r6 b081c534  r7 0000010c
01-01 08:00:22.332  1012  1012 F DEBUG   :     r8 af4f012c  r9 af2e360c  sl af2e340c  fp bebc3860
01-01 08:00:22.332  1012  1012 F DEBUG   :     ip 0000000b  sp bebc37e8  lr b052f507  pc b0531d64  cpsr 200f0010
01-01 08:00:22.340  1012  1012 F DEBUG   :    
01-01 08:00:22.340  1012  1012 F DEBUG   : backtrace:
01-01 08:00:22.341  1012  1012 F DEBUG   :     #00 pc 00049d64  /system/lib/libc.so (tgkill+12)
01-01 08:00:22.341  1012  1012 F DEBUG   :     #01 pc 00047503  /system/lib/libc.so (pthread_kill+34)
01-01 08:00:22.341  1012  1012 F DEBUG   :     #02 pc 0001d855  /system/lib/libc.so (raise+10)
01-01 08:00:22.341  1012  1012 F DEBUG   :     #03 pc 000193a1  /system/lib/libc.so (__libc_android_abort+34)
01-01 08:00:22.341  1012  1012 F DEBUG   :     #04 pc 00017014  /system/lib/libc.so (abort+4)
01-01 08:00:22.341  1012  1012 F DEBUG   :     #05 pc 0001b84f  /system/lib/libc.so (__libc_fatal+22)
01-01 08:00:22.341  1012  1012 F DEBUG   :     #06 pc 0004821b  /system/lib/libc.so (__stack_chk_fail+6)
01-01 08:00:22.341  1012  1012 F DEBUG   :     #07 pc 00002547  /system/vendor/lib/hw/hwcomposer.zx2000.so (hwc_display_dev_wait_idle+526)
01-01 08:00:22.341  1012  1012 F DEBUG   :     #08 pc 000104fc  <unknown>
```

arm-none-linux-gnueabi-addr2line -e  hwcomposer.zx2000.so  -a 00002547
