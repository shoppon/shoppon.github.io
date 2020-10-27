---
title: "镜像管理"
categories: ["云计算"]
tags: ["镜像", "服务器"]
date: 2020-10-26T10:57:00+00:00
---

# 镜像格式

| 镜像格式 | 介绍                                                         |
| -------- | ------------------------------------------------------------ |
| QCOW2    | QCOW2格式镜像是QEMU模拟器支持的一种磁盘镜像，是用一个文件的形式来表示一块固定大小的块设备磁盘。与普通的RAW格式镜像相比，QCOW2格式有如下几个特性：支持更小的磁盘占用。支持写时拷贝（CoW，Copy-On-Write），镜像文件只反映底层磁盘变化。支持快照，可以包含多个历史快照。支持压缩和加密，可以选择ZLIB压缩和AES加密。 |
| VMDK     | VMDK是VMware创建的虚拟硬盘格式。一个VMDK文件代表VMFS（云服务器文件系统）在云服务器上的一个物理硬盘驱动。 |
| VHD      | VHD是微软提供的一种虚拟硬盘文件格式。VHD文件格式可以被压缩成单个文件存放到宿主机的文件系统上，主要包括云服务器启动所需的文件系统。 |
| VHDX     | 微软在Windows Server 2012中的Hyper-V引入的一个新版本的VHD格式，称为VHDX。与VHD格式相比，VHDX具有更大的存储容量。它在电源故障期间提供数据损坏保护，并且优化了磁盘结构对齐方式，以防止新的大扇区物理磁盘性能降级。 |
| RAW      | RAW格式是直接给云服务器进行读写的文件。RAW不支持动态增长空间，是镜像中I/O性能最好的一种格式。 |
| QCOW     | QCOW通过二级索引表来管理整个镜像的空间分配，其中第二级的索引用了内存CACHE技术，需要查找动作，这方面导致性能的损失。QCOW优化性能低于QCOW2，读写性能低于RAW。 |
| VDI      | VDI是SUN公司Virtual BOX虚拟化软件所用的硬盘镜像文件格式，支持快照。 |
| QED      | QED格式是QCOW2格式的一种改进，存储定位查询方式和数据块大小与QCOW2一样。但在实现CoW（Copy-On-Write）的机制时，QED将QCOW2的引用计数表用了一个重写标记（Dirty Flag）来替代。 |

## 格式转换

`qcow2`格式和`raw`格式转换：`qemu-img convert ubuntu-20.04-server-cloudimg-amd64-disk-kvm.img -O raw ubuntu-20.04.img`

## 格式查看

查看镜像格式：`qemu-img info ubuntu-20.04-server-cloudimg-amd64-disk-kvm.img`

输出如下：

```shell
image: ubuntu-20.04-server-cloudimg-amd64-disk-kvm.img
file format: qcow2
virtual size: 2.2 GiB (2361393152 bytes)
disk size: 493 MiB
cluster_size: 65536
Format specific information:
    compat: 0.10
    refcount bits: 16
```

# 参考

- [镜像常见格式](https://support.huaweicloud.com/productdesc-ims/zh-cn_topic_0089615820.html)
- [通过qemu-img工具转换镜像格式](https://support.huaweicloud.com/bestpractice-ims/ims_bp_0030.html)

