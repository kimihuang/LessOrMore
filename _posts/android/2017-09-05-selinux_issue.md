---
layout: post
title:  selinux 问题解决方法
date:   2017-09-05 10:46:00 +0800
categories: android
tag: selinux
---

* content
{:toc}

# **SELinux问题解决方法** #

查找SELinux权限问题，一般的方法是通过查看LOG中是否有标准的SELinux Policy Exception。在Kernel Log / Main Log 中查询关键字"avc:"，如果出现以下log，则基本说明该问题就是SELinux权限问题引起的，在解决安全权限问题前，我们必须先读懂log中的信息：

    [  215.214223]<0> (0)[307:logd.auditd]type=1400 audit(1420070451.950:8): avc: denied { write } for pid=4663 comm="om.zte.engineer" 
    name="brightness" dev="sysfs" ino=9451 scontext=u:r:radio:s0 tcontext=u:object_r:sysfs:s0 tclass=file permissive=0


- * [  215.214223]：kernel time ；
-     * [307:logd.auditd]：表示此LOG 是通过auditd 打印的；
-     * type=1400  : SYSCALL ；如果type=AVC 表示为kernelevents， type=USER_AVC 表示user-space object manager events ；
-     * audit(1420070451.950:8)：audit(time:serial_number)；
-     * avc: denied { write }： field depend on what type of event is being audited.；
-     * pid=4663 comm="om.zte.engineer"：如果是个进程，pid表示为进程号，comm为运行的进程名字；
-     * scontext=u:r:radio:s0：subject context为u:r:radio:s0；
-     * tcontext=u:object_r:sysfs:s0：target context 为u:object_r:sysfs:s0；
-     * tclass : the object class of the target class=system；
-     * permissive:  permissive (1)or enforcing (0)。

上面语句的大概意思为：**om.zte.engineer这个process，使用radio的source context，访问sysfs这个文件类型时，并write文件时，被SELinux拒绝访问**。
而对于如何解决该类权限问题，一般的做法是，缺少什么就补什么，先介绍一下简化方法：
简化方法:

1. 提取所有的avc LOG.   如 adb shell "cat /proc/kmsg | grepavc" > avc_log.txt
1. 使用 **audit2allow** tool 直接生成policy. audit2allow -i avc_log.txt  即可自动输出生成的policy
1. 将对应的policy 添加到selinux policy 规则中，对应MTK Solution, 您可以将它们添加在KK: mediatek/custom/common/sepolicy, L:device/mediatek/common/sepolicy 下面，如

allow zygoteresource_cache_data_file:dir rw_dir_perms;
allow zygote resource_cache_data_file:filecreate_file_perms;
===>mediatek/custom/common/sepolicy/zygote.te (KK)
===> device/mediatek/common/sepolicy/zygote.te (L)
这样做就可以达到允许zygote对resource_cache_data_file进行create、r/w操作。
**注意**
audit2allow它自动机械的帮您将LOG 转换成policy, 而无法知道你操作的真实意图，有可能出现权限放大问题，经常出现policy 无法编译通过的情况。

如果直接按照avc: denied 的LOG 转换出SELinux Policy, 往往会产生权限放大问题. 比如因为要访问某个device, 在这个device 没有细化SELinux Label 的情况下, 可能出现:
 <7>[11281.586780] avc:  denied { read write } for pid=1217comm="mediaserver" name="tfa9897" dev="tmpfs"ino=4385 scontext=u:r:mediaserver:s0 tcontext=u:object_r:device:s0tclass=chr_file permissive=0
如果直接按照此LOG 转换出SELinuxPolicy:  allow mediaserver device:chr_file {read write};  那么就会放开mediaserver 读写所有device 的权限。而Google 为了防止这样的情况, 使用了neverallow 语句来约束, 这样你编译sepolicy 时就无法编译通过。

   下面主要介绍一下如何缺什么补什么，一步一步到没有avc: denied问题，以上面的avc log为例：

    [  215.214223]<0> (0)[307:logd.auditd]type=1400 audit(1420070451.950:8): avc: denied { write } for pid=4663 comm="om.zte.engineer" name="brightness" dev="sysfs" ino=9451 scontext=u:r:radio:s0 tcontext=u:object_r:sysfs:s0 tclass=file permissive=0 
  
**分析过程：**

- 缺少什么权限：           { write}权限；
- 谁缺少权限：                 scontext=u:r:radio:s0；
- 对哪个文件缺少权限： tcontext=u:object_r:sysfs:s0
- 什么类型的文件：        tclass=file


所以解决方法是在：radio.te中加入

**allow radio sysfs:file write;**

通过上面例子，我们可以总结出一般规律：允许某个scontext对某个tcontext拥有某个权限。
所以，可以得到一个万能套用公式：
Scontext   tcontext   tclass  avc denied权限
allow        radio         sysfs  :   file      write
有时候avc denied的log不是一次性显示所以问题，可能是要等你解决一个权限问题之后，才会提示另外一个权限问题，所以有时我们必须一次一次的试，一次一次的加。