---
title: "Openstack存储管理"
categories: ["云计算"]
tags: ["openstack", "cinder"]
date: 2020-10-28T10:49:23+08:00
typora-root-url: ../
---

# 卷

## 创建卷
从指定镜像创建卷
```shell
cinder create --image-id centos --display_name=ceph1-sys 40
```

重置挂载状态
```shell
cinder reset-state --state available --attach-status detached {volume_id}
```

## 创卷流程

拷贝镜像到卷

挂载卷

​	初始化连接

​	连接设备（卷）

​		获取可能的路径

​			发现iscsi portals

​	检查是否用

写入数据

卸载卷