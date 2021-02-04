---
title: "存储多路径相关知识"
tags: ["存储", "多路径"]
categories: ["云计算"]
date: 2020-12-22T15:30:00+08:00
typora-root-url: ../../post
---

# 多路径概念

多路径软件属于驱动程序层，一个lun通过多条链路映射到主机，会被识别成多个hdisk，多路径软件原理就是将这些hdisk整合为一个可用的盘。普通的电脑主机都是一个硬盘挂接到一个总线上，这里是一对一的关系。

配置多路径后存储上的一个LUN在主机上可以看到多个盘符(sdx/sdy)，同时还会多出一个`/dev/mapper/mpathb`存储设备，所有对硬盘的操作都应当使用这个这个设备。

多路径的主要功能就是和存储设备一起配合实现如下功能：

1. 故障的切换和恢复
2. IO流量的负载均衡
3. 磁盘的虚拟化

# 多路径配置

## 典型配置

以`EMC`为例，常见的多路径配置如下：

```shell
devices {
   device {                                    
       vendor                  "EMC"                                         //厂商名称
       product                 "CaXXXXX"                                     //产品型号
       path_grouping_policy    group_by_prio                                 //默认的路径组策略
       getuid_callout          "/sbin/scsi_id -p 0x80 -g -u -s /block/%n"    //获得唯一设备号使用的默认程序
       prio_callout            "/sbin/acs_prio_alua %d"                      //获取有限级数值使用的默认程序
       hardware_handler        "1 acs"                                       //确认用来在路径切换和IO错误时，执行特定的操作的模块。
       path_checker            hp_sw                                         //决定路径状态的方法
       path_selector           "round-robin 0"                               //选择那条路径进行下一个IO操作的方法
       failback                immediate                                     //故障恢复的模式
       no_path_retry           queue                                         //在disable queue之前系统尝试使用失效路径的次数的数值
       rr_min_io               100                                           //在当前的用户组中，在切换到另外一条路径之前的IO请求的数目
    }
}
```

使用`multipath`查询设备的failover配置和每条路径的优先级组

```shell
[root@node-13 ~]# multipath -ll
mpathbd (HUS726T4TALA600_V6J95GVV) dm-5 ATA     ,HUS726T4TALA600
size=3.6T features='0' hwhandler='0' wp=rw
`-+- policy='service-time 0' prio=1 status=active
  `- 4:0:1:0  sda   8:0    active ready running
```

# 多路径残留检测

## 残留原因

多路径残留是SAN存储一个**老大难**问题，导致残留的原因多种多样，目前已知有这几种情况：

1. 主机侧卸载卷失败。
2. 存储侧未将卷移除映射视图。

## 检测原理

当前主要通过两种手段检测多路径残留问题，一是查看`multipath -ll`输出信息中是否有`failed faulty running`；二是查看`scsi`设备的wwn号是否非法（为`-`或者特殊值`SXXX`即认为非法）。

*注：`failed faulty running`不一定代表一定是残留，也有可能确实是链路有问题（网口损坏或者线没插好）。*

## 检测流程

![multipath_check](/imgs/multipath_check.png)

当前产品实现的多路径检查逻辑如下：

1. 通过`multhpath -ll|grep 'failed faulty running'`查询是否有失败状态路径，如果有认为残留。
2. 通过`lsscsi -i`获取所有的scsi设备。
3. 获取设备类型为`disk`的所有设备。
   1. 最后一列wwn为`-`则认为是残留路径。（有一种特殊情况，如果是HUAWEI设备，倒数第二列为0则不认为是残留）
   2. 如果设备名是`/dev/xxx`，则先看其路径是否存在，不存在则不认为是残留；存在则看其wwn是不是以`['SIBM', 'SZTE', 'SFUJ', 'SINS']`开头，如果以该路径开头，则认为是残留。

当前仅支持`['IBM', 'ZTE', 'FUJITSU', '3PARdata', 'INSPUR', 'HUAWEI']`这几种厂商检测。

## 残留清理

### 判断设备是否被使用

清理残留路径前需要判断是否被使用，避免误删引起业务中断。

1. 查询链路是否被正在运行的虚拟机使用。

   ```shell
   for i in `virsh list --all|grep running|awk '{print $1}'`;do virsh dumpxml $i|grep by-id; done
   for i in `virsh list --all|grep running|awk '{print $1}'`;do virsh dumpxml $i|grep -P "by-id|instance"; done
   ```

2. 查询链路是否被已关闭的虚拟机使用。

   ```shell
   grep -rn "by-id" /var/lib/nova/|grep ${path}
   ```

3. 查询链路是否仍在系统中使用。

   ```shell
   dmsetup info /dev/disk/by-id/dm-uuid-mpath-xxxxxx
   ```

   如果`count`为0则认为未被使用。

### 刷新多路径

刷新残留的多路径

```shell
multipath -f ${mpath}
```

### 移除设备

查看路径有没有被虚拟机使用

```shell
[root@node-38 ~]# dmsetup info /dev/disk/by-id/dm-uuid-mpath-3600a09803830456c463f4c585a545147
Name:              3600a09803830456c463f4c585a545147
State:             ACTIVE
Read Ahead:        256
Tables present:    LIVE
Open count:        7
Event number:      40
Major, minor:      253, 5
Number of targets: 1
UUID: mpath-3600a09803830456c463f4c585a545147
```

如果`open_count`为0非则代表未被使用，可使用以下命令清理

```shell
dmsetup remove /dev/disk/by-id/dm-uuid-mpath-360xxx
multipath -f /dev/disk/by-id/dm-uuid-mpath-360xxx
```

如果`open_count`不为0即为IO占用，不能清理。

### 删除scsi设备

```shell
echo 1 > /sys/block/${sdX}/device/delete
```

# 参考

- [LINUX下多路径（multi-path）介绍及使用](https://blog.51cto.com/rootking/476212)
- [Linux配置和管理设备映射多路径multipath](https://www.cnblogs.com/lijiaman/p/13893346.html)
- [配置multipath.conf](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/dm_multipath/config_file_devices)
- [MultipathUsageGuide](http://www.sourceware.org/lvm2/wiki/MultipathUsageGuide)
- [Linux multipath多路径报错故障处理](https://blog.csdn.net/qq_33211463/article/details/106783397)
- [How to Scan/Detect New LUNs on Linux](https://linoxide.com/storage/scandetect-luns-redhat-linux-outputs-remember/)
- [SCANNING STORAGE INTERCONNECTS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/online_storage_reconfiguration_guide/scanning-storage-interconnects)
- [移除多路径设备](https://elkano.org/blog/removing-multipath-device/)
- [计算节点多路径残留](https://easystack.atlassian.net/wiki/spaces/ESK/pages/85950818/multipath)

