---
title: "iSCSI原理及使用"
tags: ["存储"]
categories: ["云计算"]
date: 2020-12-22T19:30:00+08:00
typora-root-url: ../
---

# iSCSI原理

iSCSI（互联网小型计算机系统接口）是一种在TCP/IP上进行数据块传输的标准。它是由Cisco和IBM两家发起的，并且得到了各大存储厂商的大力支持。iSCSI可以实现在IP网络上运行SCSI协议，使其能够在诸如高速千兆以太网上进行快速的数据存取备份操作。

iSCSI利用了TCP/IP的port 860 和 3260 作为沟通的渠道。透过两部计算机之间利用iSCSI的协议来交换[SCSI](https://zh.wikipedia.org/wiki/SCSI)命令，让计算机可以透过高速的局域网集线来把**SAN模拟成为本地的储存设备**。

其大体架构如下：

![iscsi_arch](/imgs/iscsi_arch.png)

# iSCSI概念

## 名词

### 启动器

用于访问 iSCSI 存储的客户端称为启动器。此 iSCSI 启动器可以连接到服务器（iSCSI 目标）。在这种情况下，iSCSI 启动器会将 SCSI 命令发送到 iSCSI 目标。出于此目的，这些 SCSI 命令会在 IP 数据包中打包。

### 目标器

iSCSI 目标设备可接收 iSCSI 命令并共享存储。该存储可以是物理磁盘，也可以是表示多个磁盘的区域或物理磁盘的一部分。存储阵列是典型的 iSCSI 目标。

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
tgtadm --lld iscsi --mode target --op bind --tid 1 --initiator-address=x.x.x.x
```

## 客户端

发现目标器

```shell
iscsiadm --mode discovery --type sendtargets --portal x.x.x.x
```

登陆目标器

```shell
iscsiadm -m node -T iqn.2015-05-04.org.tecstack.storage.tg1 --login
```

查看连接

```shell
iscsiadm –m session
```

重新扫描iscsi设备

```shell
iscsiadm –m node –R
echo '---' > /sys/class/scsi_host/host0/scan
```

# Ceph对iscsi的支持

## 设计及依赖

一般而言，Ceph集群块存储被限制为通过`librbd`被`qemu`使用。从`Luminous`开始，块存储开始支持标准的`iSCSI`访问以支持更广泛的平台，需要满足以下条件：

1. Red Hat Enterprise Linux/CentOS 7.5 (or newer); Linux kernel v4.16 (or newer)
2. iSCSI Gateway节点，可以和OSD节点同时部署或者使用专有节点
3. iSCSI前端流量和Ceph后端流量使用独立的网络

Ceph iSCSI Gateway即是一个iSCSI目标，又是一个Ceph客户端；可以将其理解成Ceph RBD接口与标准iSCSI协议之间的**”翻译器“**。

## 使用步骤

Ceph iSCSI Gateway的主要使用步骤如下：

1. 安装iSCSI软件

   ```shell
   yum install ceph-iscsi
   yum install tcmu-runner
   ```
   
2. 配置网关并启动

   ```shell
   touch /etc/ceph/iscsi-gateway.cfg
   
   systemctl enable rbd-target-gw
   systemctl start rbd-target-gw
   
   systemctl enable rbd-target-api
   systemctl start rbd-target-api
   ```

3. 创建目标并添加卷到客户端

   ```shell
   gwcli
   create iqn.2003-01.com.redhat.iscsi-gw:iscsi-igw
   cd iqn.2003-01.com.redhat.iscsi-gw:iscsi-igw/gateways
   create ceph-gw-1 10.172.19.21
   create ceph-gw-2 10.172.19.22
   cd /disks
   create pool=rbd image=disk_1 size=90G
   cd /iscsi-target/iqn.2003-01.com.redhat.iscsi-gw:iscsi-igw/hosts
   create iqn.1994-05.com.redhat:rh7-client
   disk add rbd/disk_1
   ```

# OpenStack对接iSCSI存储

## 创建iscsi卷流程

 cinder创建iscsi卷过程中会调用driver进行映射，然后通过`os-brick`进行扫盘获取盘符，然后将镜像数据下载到磁盘中。具体流程如下：

![iscsi_flow](/imgs/iscsi_flow.png)

各组件职责如下：

* Cinder volume：任务编排。
* Osbrick：连接iscsi并扫盘。
* Cinder driver：创建卷和映射视图。
* IPSAN：提供创建、映射等相关接口。

## 实现接口列表

存储使用`iSCSI`方式对接openstack时，除了需要实现基础的`create_volume`、`create_snapshot`、`delete_volume`等接口之外，还需要额外实现以下接口，实现`initialize_connection`和`terminate_connection`接口实现卷的映射和解映射。

```python
def initialize_connection(self, volume, connector):
  pass

def terminate_connection(self, volume, connector, **kwargs):
  pass
```

# ESS支持iSCSI需求

## 管控面

一般来说，如果将ceph作为openstack存储后端，推荐使用`rbd`方式对接。

### 提供cinder iscsi driver

提供cinder driver，实现initialize_connection和terminate_connection接口，调用ESS接口进行卷映射。

### 支持iSCSI方式对接ESS

提供商业存储对接包，以标准IPSAN方式对接ESS。

## 数据面

### 提供映射卷相关接口

参考其他友商存储

# 参考

- [使用Linux的tgtd提供iscsi服务](http://promisejohn.github.io/2015/06/04/iSCSIwithTgtd/)
- [RBD vs. iSCSI: How best to connect your hosts to a Ceph cluster](https://www.suse.com/c/rbd-vs-iscsi-how-best-to-connect-your-hosts-to-a-ceph-cluster/)
- [Ceph iSCSI gateway](https://docs.ceph.com/en/latest/rbd/iscsi-overview/)

