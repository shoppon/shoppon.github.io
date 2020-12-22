---
title: "存储多路径相关知识"
tags: ["存储", "多路径"]
categories: ["云计算"]
date: 2020-12-22T15:30:00+08:00
---

# 多路径概念

配置多路径后存储上的一个LUN在主机上可以看到多个盘符

# 多路径配置

## 使用lsscsi查看scsi设备

使用`-s`参数查询设备大小，`-d`参数查询设备节点名称，`-i`参数查询wwn

```shell
[root@node-21 ~]# lsscsi -i -d -s
[7:0:0:0]    process Marvell  Console          1.01  -          -       -
[8:0:0:0]    cd/dvd  Byosoft  Virtual CDROM0   1.00  /dev/sr0 [11:0]  Byosoft_Virtual_CDROM0_AAAABBBBCCCC1-0:0       -
[9:0:10:0]   disk    TOSHIBA  AL14SEB120N      1402  /dev/sda [8:0]  350000399e8510fcd  1.20TB
[9:0:11:0]   disk    TOSHIBA  AL14SEB120N      1402  /dev/sdb [8:16]  350000399e85105a5  1.20TB
[9:0:12:0]   disk    TOSHIBA  AL14SEB120N      1402  /dev/sdc [8:32]  350000399e8510ff1  1.20TB
[9:0:13:0]   disk    TOSHIBA  AL14SEB120N      1402  /dev/sdd [8:48]  350000399e8510755  1.20TB
[9:2:0:0]    disk    AVAGO    MR9361-8i        4.68  /dev/sde [8:64]  3600605b00f767c402675324ac4402b1e   479GB
[9:2:1:0]    disk    AVAGO    MR9361-8i        4.68  /dev/sdf [8:80]  3600605b00f767c402679c5910fea398c   599GB
[12:0:0:2]   disk    IBM      2145             0000  /dev/sdk [8:160]  -       -
[13:0:0:2]   disk    IBM      2145             0000  /dev/sdl [8:176]  SIBM_2145_00c02082fbaeXX25       -
```

## 使用scsi_id获取设备标识符号

使用`-u`参数替换空白符

```shell
[root@node-21 ~]# /lib/udev/scsi_id -u -g -d /dev/sdd
350000399e8510755
```

# 常见问题

## 多路径残留问题

当系统中多路径相关配置出现问题时，可能会导致主机上多路径设备出现遗留。

清理残留的scsi设备

```shell
echo 1 > /sys/block/sdX/device/delete
```

清理残留的多路径

```shell
multipath -f ${mpath}
```

