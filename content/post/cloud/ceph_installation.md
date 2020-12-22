---
11title: "Ceph安装部署"
categories: ["云计算"]
tags: ["ceph", "存储"]
date: 2020-12-05T16:50:00+08:00
---

# 安装dashboard

安装插件包

```shell
yum install -y ceph-mgr-dashboard -y
```

开启dashboard插件

```shell
ceph mgr module enable dashboard
```

禁用SSL

```shell
ceph config set mgr mgr/dashboard/ssl false
```

配置监听IP及端口

```shell
ceph config set mgr mgr/dashboard/server_addr 0.0.0.0
ceph config set mgr mgr/dashboard/server_port 8080
```

服务管理

```shell
ceph mgr services
ceph mgr module disable dashboard
ceph mgr module enable dashboard
```



# 参考

- [Ceph Dashboard全功能安装集成](https://zhuanlan.zhihu.com/p/135288558)
- [手动安装iscsi target cli](https://docs.ceph.com/en/latest/rbd/iscsi-target-cli-manual-install/)
- [安装librados](https://docs.ceph.com/en/latest/rados/api/librados-intro/#getting-librados-for-python)

