---
title: "iSCSI原理及使用"
tags: ["存储"]
categories: ["云计算"]
date: 2020-12-22T19:30:00+08:00
typora-root-url: ../
---

# iSCSI原理

iSCSI（互联网小型计算机系统接口）是一种在TCP/IP上进行数据块传输的标准。它是由Cisco和IBM两家发起的，并且得到了各大存储厂商的大力支持。iSCSI可以实现在IP网络上运行SCSI协议，使其能够在诸如高速千兆以太网上进行快速的数据存取备份操作。

iSCSI利用了TCP/IP的port 860 和 3260 作为沟通的渠道。透过两部计算机之间利用iSCSI的协议来交换[SCSI](https://zh.wikipedia.org/wiki/SCSI)命令，让计算机可以透过高速的局域网集线来把SAN模拟成为本地的储存设备。

# iSCSI概念

## 名词

### 启动器

### 目标器

# iSCSI使用流程

iSCSI总体使用流程如下：

![iscsi](/imgs/iscsi.png)

## 服务端

添加`target`

```shell
tgtadm --lld iscsi --op new --mode target --tid 1 --targetname iqn.2015-05-04.org.tecstack.storage.tg1
```

添加`backstorage`

```shell
tgtadm --lld iscsi --op new --mode logicalunit --tid 1 --lun 2 --backing-store disk1.img
```

绑定客户端IP

```shell
tgtadm --lld iscsi --mode target --op bind --tid 1 --initiator-address=192.168.182.157
```

## 客户端



# OpenStack对接iSCSI存储

## 实现接口列表

# 参考

- [使用Linux的tgtd提供iscsi服务](http://promisejohn.github.io/2015/06/04/iSCSIwithTgtd/)
- [RBD vs. iSCSI: How best to connect your hosts to a Ceph cluster](https://www.suse.com/c/rbd-vs-iscsi-how-best-to-connect-your-hosts-to-a-ceph-cluster/)

