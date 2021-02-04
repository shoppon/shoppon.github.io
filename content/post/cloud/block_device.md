---
title: "块设备相关知识"
tags: ["存储", "块设备"]
categories: ["云计算"]
date: 2021-01-28T16:30:00+08:00
typora-root-url: ../../post
---

# 块设备概念

# 常用操作

## 查询所有块设备

块设备是也是一种特殊文件，可通过`ls -l`查询所有块设备。

```shell
ls -l /dev/disk/by-path/
```

## 检查设备是否可用

可使用`dd`命令检查设备是否可用。

```shell
dd if=${path} of=/dev/null count=1
```

如果有速率输出则认为是可用。

```shell
root@ubuntu:~# dd if=/dev/vda of=/dev/null count=1
1+0 records in
1+0 records out
512 bytes copied, 7.2674e-05 s, 7.0 MB/s
```

## 查询块设备是否被虚拟机使用

查看虚拟机的xml配置文件中是否包含块设备的wwn。

```shell
for i in `virsh list --all|grep running|awk '{print $1}'`;do virsh dumpxml $i|grep by-id; done
```

## 查询是否被已关闭虚拟机使用

```shell
grep -rn "by-id" /var/lib/nova/
```

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

## 扫盘

使用`echo`命令扫盘

```shell
echo "- - -" >  /sys/class/scsi_host/host0/scan
echo "c t l" >  /sys/class/scsi_host/hosth/scan
```

- `h` 代表hba卡编号
- `c` 代表hba卡通道channel
- `t` 代表SCSI目标ID
- `l` 代表LUN

查询HBA卡编号

```shell
ls /sys/class/scsi_host
```

扫描所有设备所有LUN

```shell
# for host in `ls /sys/class/scsi_host/`;do
echo "- - -" >/sys/class/scsi_host/${host}/scan;
done
# for host in `ls /sys/class/fc_host/`; do
echo "1" >/sys/class/fc_host/${host}/issue_lip;
done
```

## 移除块设备

使用`echo`进行移除。

```shell
echo 1 > /sys/block/${sdX}/device/delete
```

## 强制清理设备

```shell
dmsetup clear /dev/disk/by-id/dm-uuid-mpath-360xxx
dmsetup wipe_table /dev/disk/by-id/dm-uuid-mpath-360xxx
dmsetup remove /dev/disk/by-id/dm-uuid-mpath-360xxx
```

# 参考

* [移除多路径设备](https://elkano.org/blog/removing-multipath-device/)
* [计算节点多路径残留](https://easystack.atlassian.net/wiki/spaces/ESK/pages/85950818/multipath)

