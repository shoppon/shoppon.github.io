---
title: "灾备竞争分析"
categories: ["灾备"]
tags: ["私有云", "灾备"]
date: 2020-11-13T10:01:23+08:00
typora-root-url: ../../
---

# 友商产品简介

## 私有云厂商

### VMware

产品名称：VMware vCloud suite

研发实力最强，拥有服务器虚拟机vSphere、存储虚拟机vSAN、网络虚拟机NSX全栈技术。

提供**云中实验室**体验所有云化产品。

### 华为

产品名称：HuaweiCloudStack

主要特性：VolumeBackupService、CloudServerBackupService、VolumeHighAvailable、CloudServerHighAvailable

### H3C

产品名称：H3C CloudOS

H3C无独立自研的灾备产品，主要走合作路线，与爱数、Commvault、凌云动力等厂商深度合作打造一体化的备份、IBM小机管理等解决方案。

只支持全量备份，系统将源云硬盘的文件进行复制，以云硬盘文件的形式存放在对应存储可用域的存储池中；

产品名称：H3C UniStor CB

[产品文档](http://www.h3c.com/cn/Products___Technology/Products/Storage/Catalog/H3C/Data_Backup/H3C_UniStor_CB/#)

[云硬盘产品手册](http://www.h3c.com/cn/pub/document_center/2019/cloudos/webhelp/webhelp/service/volume/volumeList.html)

[虚拟机备份操作指南](http://www.h3c.com/cn/pub/Document_Center/2020/03/H3C_CloudOS_5.0WebHelp/html/topic_m200415x1_1905.htm#_Ref493595273)

### 浪潮

产品名称：浪潮

主要特性：

云服务器备份：通过对云服务器系统盘和所有挂载的云硬盘做一致性快照，实现云服务器的备份功能，当云服务器自身或内部数据遇到错误时，可以从备份快速恢复整个云服务器，最大程度上缩短业务中断时间。

云硬盘备份：依靠云硬盘增量快照功能，实现盘内数据的快速备份和恢复，恢复云硬盘时可以覆盖原硬盘，也可以恢复为新的云硬盘来恢复某一个文件。

文件备份：通过对虚拟机内安装备份客户端软件，可以对用户核心的文件或文件夹的进行备份，与云硬盘和云服务器相比可以大幅度节省备份存储成本。

[云备份服务介绍文档](https://console1.cloud.inspur.com/document/cbs/1-service-introduction.html)

[混合云备份HBR](https://console1.cloud.inspur.com/document/hbr/index.html)

## 公有云厂商

### 阿里云

产品名称：Apsara Stack

ASR-DR(Apsara Stack Resilience Disaster Recovery)是一款提供容灾功能的云平台产品，当前支持 OSS、RDS、Tablestore、DRDS、DNS和MQ的热备容灾。

异地备份控制台是一款专注于异地备份与恢复的云平台产品，当前支持RDS、OSS、ECS、NAS的备份恢复。

### AWS

产品名称：AWS Backup

集中配置备份策略并监控 Amazon EBS 卷、Amazon EC2 实例、Amazon RDS 数据库、Amazon DynamoDB 表、Amazon EFS 文件系统和 AWS Storage Gateway 卷等 AWS 资源的备份活动。

[AWS Backup 功能](https://aws.amazon.com/cn/backup/features/)

产品名称：CloudEndure Disaster Recovery

数据中心故障、服务器损坏或网络攻击等 IT 灾难不仅会扰乱您的业务，还会造成数据丢失、影响您的收入并损害您的声誉。CloudEndure Disaster Recovery 可以通过快速而可靠地将物理、虚拟和基于云的服务器恢复到 AWS，最大限度地缩短停机时间并减少数据丢失。

[CloudEndure DR功能介绍](https://aws.amazon.com/cn/cloudendure-disaster-recovery/?nc2=h_ql_prod_st_cedr)

### Azure

产品名称：Azure 备份

Azure 备份是一个经济高效、安全、一键式备份解决方案，可根据备份存储需求进行缩放。借助集中式管理界面，可轻松定义备份策略和保护多种企业工作负载，包括 Azure 虚拟机、SQL 和 SAP 数据库以及 Azure 文件共享。

该框架提供一个选项，用于在创建 VM 快照时运行自定义的操作前脚本和操作后脚本。 前脚本在创建 VM 快照前的那一刻运行，后脚本在创建 VM 快照后立即运行。 使用前脚本和后脚本可在创建 VM 快照时灵活控制应用程序和环境。

前脚本可以调用本机应用程序 API 来使 IO 处于静默状态并将内存中内容刷新到磁盘。 这些操作可确保快照是应用程序一致的。 后脚本使用本机应用程序 API 来解冻 IO，使应用程序能够在创建 VM 快照后恢复正常操作。

[Azure备份文档](https://azure.microsoft.com/zh-cn/services/backup/)

[Azure应用一致性实现方法](https://docs.microsoft.com/zh-cn/azure/backup/backup-azure-linux-app-consistent)

产品名称：Azure Site Recovery

Recovery Plan定义机器如何切换以及切换的顺序，一个恢复计划最多支持100个机器。

使用Recovery Plan的好处：

- 定义app之间的依赖
- 自动恢复任务降低RTO
- 保证应用是灾难恢复计划的一部分
- 通过容灾演练确保灾难恢复达到期望。

[ASR文档](https://docs.microsoft.com/zh-cn/azure/site-recovery/site-recovery-overview)

## 存储厂商

### XSKY

产品名称：X3DS立体数据管理系统

主要特性：仅提供**非结构化数据**的复制、迁移、备份、归档功能。

### 杉岩

产品名称：SandStone HyperCube

SandStone HyperCube可以无缝对接阿里云等公有云平台，通过统一管理视图实现多云管理。

## 超融合厂商

### Nutanix

产品名称：Leap灾难恢复

设置灾难恢复解决方案从未如此简单，无需安装任何东西。从 Prism 启动和管理 Xi Leap 灾难恢复服务，并利用该界面管理 Nutanix AOS 集群。对您的虚拟机进行分类，并利用保护策略和恢复计划来定制您的环境。使用相同的管理界面来启动计划内或计划外的故障切换，并在非生产子网上进行实时灾难恢复测试。

[基于云的灾难恢复解决方案](https://www.nutanix.com/cn/products/leap)

# 特性对比

| 特性           | ECS  | VMware | 华为Stack | H3C  | 浪潮 | XSKY | 杉岩 | Nutanix | 阿里 | AWS  | Azure |
| -------------- | ---- | ------ | --------- | ---- | ---- | ---- | ---- | ------- | ---- | ---- | ----- |
| 块备份         | ❌    | ✔️      | ✔️         | ❌    | ✔️    | ❌    | NA   | ❌       | ✔️    | ✔️    | ✔️     |
| 文件备份       | ❌    | ❌      | ❌         | ❌    | ✔️    | ❌    | NA   | ❌       | ✔️    | ✔️    | ✔️     |
| 对象备份       |      | ❌      | ❌         | ❌    | ❌    | ✔️    | NA   | ❌       | ❌    | ✔️    | ❌     |
| 应用备份       |      | ❌      | ✔️         | ❌    | ❌    | ❌    | NA   | ❌       | ✔️    | ✔️    | ✔️     |
| 混合云备份     |      | ❌      | ✔️         | ❌    | ✔️    | ❌    | ✔️    | ❌       | ✔️    | ✔️    | ✔️     |
| 主机容灾       |      | ❌      | ✔️         | ❌    | ❌    | ❌    | NA   | ✔️       | NA   | ✔️    | ✔️     |
| 应用容灾       |      | ❌      | ❌         | ❌    | ❌    | ❌    | NA   | ✔️       | ✔️    | ✔️    | ❌     |
| 站点容灾       |      | ❌      | ❌         | ❌    | ❌    | ❌    | NA   | ✔️       | ✔️    | NA   | ✔️     |
| 两地三中心     |      | ❌      | ✔️         | ❌    | ❌    | ❌    | NA   | ❌       | ✔️    | ❌    | ❌     |
| 混合云容灾     |      | ✔️      | ❌         | ❌    | ❌    | ❌    | NA   | ✔️       | ✔️    | ✔️    | ✔️     |
| 对象跨区域复制 |      | ❌      | ✔️         | ❌    | ❌    | ✔️    | NA   | ❌       | ✔️    | ✔️    | ✔️     |

# 概况

3A的备份、容灾特性最全面，块、文件、应用、混合云几乎全部支持。

国内厂商的研发能力普遍较弱，H3C仅和爱数、CV等备份厂商合作，未提供容灾方案；XKSY只支持非结构化数据的复制；浪潮仅有简单的块、文件和混合云备份。

VMware、Nutanix、Azure、AWS等国外先进厂商都提供了基于云的灾难恢复解决方案，国内的杉岩也可以对接阿里公有云。

# 建议

容灾：对标VMware和Azure构建跨云容灾能力，**Run ECS Cross Cloud**。	

备份：基于ceph snapshot封装最基本的**块备份**能力，应用备份、文件备份等高级特性不深入touch。

# 参考

- [Gartner：2019 年 DRaaS 魔力象限](http://www.ctiforum.com/news/guonei/558277.html)