---
layout: post
title:  fstab  mount 路径
date:   2017-09-05 16:50:00 +0800
categories: android
tag: init
---

* content
{:toc}

使用emmc启动android系统时，android会把系统分区mount到ramkdisk文件系统的指定目录
,ramdisk里面的fstab.emmc文件会存放需要mount的分区的路径及mount点，如下：
kimihuang@kimihuang-Z87X-D3H:root$ cat fstab.emmc 

    #Android fstab file.
	#<src>  <mnt_point> <type><mnt_flags and options>   <fs_mgr_flags>
	#The filesystem that contains the filesystem checker binary (typically /system) cannot
	#pecify MF_CHECK, and must come before any filesystems that do specify MF_CHECK
    /dev/block/platform/soc/d800a000.emmc/by-name/system/system ext4   ro,barrier=1   wait
    /dev/block/platform/soc/d800a000.emmc/by-name/cache /cache  ext4   noatime,nosuid,nodev,barrier=1,data=orderedwait,check
    /dev/block/platform/soc/d800a000.emmc/by-name/userdata  /data   ext4   noatime,nosuid,nodev,barrier=1,data=ordered,noauto_da_allocwait,check
    /dev/block/platform/soc/d800a000.emmc/by-name/misc  /misc   emmc   defaults   defaults
    /dev/block/platform/soc/d800a000.emmc/by-name/boot  /boot   emmc   defaults   defaults
    /dev/block/platform/soc/d800a000.emmc/by-name/recovery  /recovery   emmc   defaults   defaults
不同android版本的系统可能新建不同的节点，比如：

**android 5.1: /dev/block/platform/soc/by-name/system**

**android 7.0: /dev/block/platform/soc/d800a000.emmc/by-name/system**

这些by-name的节点是在android的init进程里面创建的，具体的创建文件在devices.c 或者devices.cpp.

**by-name就在下面这段代码里**

	static char **get_block_device_symlinks(struct uevent *uevent)
	{
	    const char *device;
	    struct platform_node *pdev;
	    char *slash;
	    const char *type;
	    char buf[256];
	    char link_path[256];
	    int link_num = 0;
	    char *p;
	
	    pdev = find_platform_device(uevent->path);
	    if (pdev) {
	        device = pdev->name;
	        type = "platform";
	    } else if (!find_pci_device_prefix(uevent->path, buf, sizeof(buf))) {
	        device = buf;
	        type = "pci";
	    } else {
	        return NULL;
	    }
	
	    char **links = (char**) malloc(sizeof(char *) * 4);
	    if (!links)
	        return NULL;
	    memset(links, 0, sizeof(char *) * 4);
	
	    INFO("found %s device %s\n", type, device);
	
	    snprintf(link_path, sizeof(link_path), "/dev/block/%s/%s", type, device);
	
	    if (uevent->partition_name) {
	        p = strdup(uevent->partition_name);
	        sanitize(p);
	        if (strcmp(uevent->partition_name, p))
	            NOTICE("Linking partition '%s' as '%s'\n", uevent->partition_name, p);
	        if (asprintf(&links[link_num], "%s/**by-name**/%s", link_path, p) > 0)
	            link_num++;
	        else
	            links[link_num] = NULL;
	        free(p);
	    }
	
	    if (uevent->partition_num >= 0) {
	        if (asprintf(&links[link_num], "%s/by-num/p%d", link_path, uevent->partition_num) > 0)
	            link_num++;
	        else
	            links[link_num] = NULL;
	    }
	
	    slash = strrchr(uevent->path, '/');
	    if (asprintf(&links[link_num], "%s/%s", link_path, slash + 1) > 0)
	        link_num++;
	    else
	        links[link_num] = NULL;
	
	    return links;
	}

