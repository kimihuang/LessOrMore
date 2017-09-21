---
layout: post
title:  OpenSSH移植到ARM Linux教程
date:   2017-09-14 08:08:00 +0800
categories: tools
tag: OpenSSH移植到ARM Linux教程
---

* content
{:toc}

制作openssh 可以详细步骤参考<http://www.veryarm.com/892.html>

下面是本地制作过程，稍微有点点区别
## 1，下载源码
Zlib 官方下载：<http://www.zlib.net/>

Openssl官方下载： <http://www.openssl.org/source>

Openssh 官方下载： <http://www.openssh.com/portable.html>

 下载的版本依次为：zlib-1.2.8.tar.gz、openssl-1.0.2j.tar.gz、openssh-6.6p1.tar.gz
## 2, 由于移植涉及三个包的编译，最好建立工作目录
在~/目录下新建ssh 目录
 Ssh目录下 新建三个文件夹
 分别为：compressed 用于放压缩包  install 用于放安装包  source 用于放置压缩包解压后的源文件
 解压：
 ```
 $ cd compressed
 $ tar zxvf zlib-1.2.8.tar.gz  -C ../source
 $ tar zxvf openssl-1.0.2.j.tar.gz  -C ../source
 $ tar zxvf openssh-6.6p1.tar.gz  -C ../source
 ```
## 3, 编译
 交叉编译zlib
 编译zlib为动态 .so
 ```
 $ cd ~/ssh/source/ zlib-1.2.8
 $ prefix=~/ssh/install/ zlib-1.2.8 CC=arm-none-linux-gnueabi-gcc AR= arm-none-linux-gnueabi-ar ./configure
 $ make
 $ make install
 ```

 编译zlib为静态 .a
 ```
 $ cd ~/ssh/source/ zlib-1.2.8
 $ vi configure  
 将line:69行： shared=1   ->    shared=0
 $ prefix=~/ssh/install/ zlib-1.2.8 CC=arm-none-linux-gnueabi-gcc AR= arm-none-linux-gnueabi-ar ./configure
 $ make
 $ make install
 ```

 交叉编译 openssl
 ```
 $ cd ~/ssh/source/ openssl-1.0.2.j
 $ ./Configure --prefix=~ /ssh/install/ openssl-1.0.2.j  os/compiler:arm-none-linux-gnueabi-gcc
 $ make
 $ make install
 ```

 交叉编译openssh
 ```
 $ cd ~/ssh/source/ openssh-6.6p1
 $ ./configure --host=arm-none-linux-gnueabi --with-libs --with-zlib=~/ssh/install/ zlib-1.2.8 --with-ssl-dir=~ /ssh/install/ openssl-1.0.2.j --disable-etc-default-login CC=arm-none-linux-gnueabi-gcc AR=arm-none-linux-gnueabi-ar
 $ make
 ```

 将编译openssh生成的下列文件拷贝到目标板文件系统的相应位置
scp、sftp、ssh sshd、ssh-add、ssh-agent、ssh-keygen、ssh-keyscan共8个文件拷贝到目标板/usr/local/bin
moduli、ssh_config、sshd_config共3个文件拷贝到目标板 /usr/local/etc
sftp-server、ssh-keysign 共2个文件拷贝到目标板 /usr/local/libexec
 ## 4, 生成Key文件
在pc端执行下面命令，把生成的key文件复制到目标板/usr/local/etc/ 目录
```
$ ssh-keygen -t rsa -f ssh_host_rsa_key -N ""
$ ssh-keygen -t dsa -f ssh_host_dsa_key -N ""
$ ssh-keygen -t ecdsa -f ssh_host_ecdsa_key -N ""
$ ssh-keygen -t dsa -f ssh_host_ed25519_key -N ""
```

修改 ssh_host_ed25519_key 权限为 600：
```
$ chmod 600 ssh_host_ed25519_key
```
## 5，目标板用户信息
   首先检查目标板系统/etc 目录下是否有passwd文件，如果没有，可以从pc 拷贝/etc/passwd 到目标板的 /etc目录
   修改目标板系统的passwd文件
   ```
   $ vi etc/passwd
   root:x:0:0:root:/root:/bin/bash   ->     root:x:0:0:root:/root:/bin/sh
   ```
   添加一行：
   ```
   sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
   ```
## 6,  启动目标板系统测试
   如果没有设置密码，首先需要设置密码
   ```
   # passwd root
   ```

   执行
   ```
   # /usr/local/bin/sshd
   ```
   如果运行的过程中有提示缺少动态连接库，可以在主机上搜索相应文件，拷贝到目标板/lib/目录下面，注意创建软连接！，
如果使用静态库，则不会出现缺少动态库。可能出现的错误上面的网址有补充说明。

此时，可以从主机执行
```
ssh root@**.**.**.**  
```
连接目标板






***

Ramdisk 文件修改如下，你可以直接把U盘里的文件拷贝到ramdisk里。
1，	新建文件夹
```
/usr/local/bin
/usr/local/etc
/usr/local/libexec
/var/run     //权限root:root
/var/empty  //权限 root:root
/tmp
/dev/pts
```
2，	添加文件
```
/ip.sh                  //主要用于配置获取ip地址
/etc/init.d/ifconfig-eth0    

/etc/passwd            //密码
/usr/local/bin     目录所有文件
/usr/local/etc     目录所有文件
/usr/local/libexec  目录所有文件
```

3, 修改
文件/etc/init.d/rcS
mount devpts /dev/pts -t devpts    //mount devpts文件系统，主机登录:  ssh root@\*\*.\*\*.\*\*.\*\*  需要devpts文件系统
./ip.sh &             //如果不需要获取ip,可以不执行这条命令
/usr/local/bin/sshd    //执行sshd

4，由于我编译zlib时，选择的是静态编译，因此没有动态链接库，不需要再目标板lib下放置动态链接库。
