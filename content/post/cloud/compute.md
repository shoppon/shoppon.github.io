---
title: "Openstack计算管理"
categories: ["云计算"]
tags: ["nova", "compute"]
date: 2020-10-27T10:49:23+08:00
typora-root-url: ../
---

# 常用命令
## 创建虚拟机
使用`cloud-init`注入root用户密码，将以下内容保存为`user_data`文件：
```shell
#!/bin/sh
passwd root<<EOF\n
222333\n
222333\n
EOF
```
创建虚拟机时指定卷、flavor、密钥文件、cloud-init文件、网络。
```shell
openstack server create --volume ceph3-sys --flavor 4U4G --key-name mac --user-data user_data --network internal_network ceph3
```
## 重置虚拟机状态
```shell
nova reset-state --active ubuntu
```

## flavor
flavor创建后无法修改
```shell
openstack flavor create --vcpus 4 --ram 4096 --public 4U4G
```