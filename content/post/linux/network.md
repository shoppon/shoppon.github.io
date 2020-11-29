---
title: "网络相关"
categories: ["linux"]
tags: [""]
date: 2020-05-16T17:35:58+08:00
---

# 网络相关

## 交换机

### 三层网络模型

**接入层：**为终端客户提供接入功能。

**汇聚层：**连接网络的核心层和各个接入的应用层，采用三层交换机。二层网络通过MAC地址实现通讯，提供基于策略的连接。

**核心层：**网络高速交换主干，通过IP路由实现跨网段通讯。

## 工具

## netstat

| Proto | Recv-Q | Send-Q | Local Address       | Foreign Address    | State       | PID/Prog |
| :---- | ------ | ------ | ------------------- | ------------------ | ----------- | -------- |
| tcp   | 0      | 0      | 192.168.0.141:29210 | 0.0.0.0:*          | LISTEN      | 7584/oma |
| tcp   | 0      | 28336  | 192.168.0.141:48658 | 10.246.160.228:443 | ESTABLISHED | 7584/oma |

### iptables

清空所有iptables

```shell
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
```

### BBR

查看当前系统流控策略：`sysctl net.ipv4.tcp_available_congestion_control`

开启BBR方式：

```shell
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

查看bbr是否启动：`lsmod | grep bbr`

###  ethtool

使用ethtool查看网卡详情，包括速率、是否连接、双工模式等。