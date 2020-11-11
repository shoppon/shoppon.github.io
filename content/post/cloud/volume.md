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