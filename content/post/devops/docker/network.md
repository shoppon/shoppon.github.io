---
title: "容器网络"
categories: ["docker"]
tags: [""]
date: 2020-07-11T15:31:32+08:00
---

# 容器网络

**查询网络列表：**`docker network ls`

**模拟断网：**`docker network disconnect [OPTIONS] NETWORK CONTAINER`

**模拟连网：**`docker network connect [OPTIONS] NETWORK CONTAINER`

### 实战

- 断开sdra网络`docker network disconnect docker_default docker_soma_1`
- 连接sdra网络`docker network connect docker_default docker_soma_1`

## traffic control

### 安装

执行`apt-get install iproute2`安装。

### 操作

**设置200ms延迟：**`tc qdisc add dev eth0 root netem delay 200ms`
