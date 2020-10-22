---
title: "系统管理"
categories: ["linux"]
tags: [""]
date: 2020-06-11T21:25:31+08:00
---

```
service sysstat restart 
```

# 系统管理

### 系统性能统计sar

sar是system activity reporter的缩写。

每隔2秒刷新网络流程：`sar -n DEV 2`

每隔2秒查看CPU使用情况5次：`sar 2 5`

## 内存

使用`free`命令查看内存使用情况，各字段含义如下：

| **名字** | **含义**                                       |
| -------- | ---------------------------------------------- |
| VIRT     | 进程使用的虚拟内存总量，VIRT=SWAP+RES          |
| SWAP     | 进程使用的虚拟内存中，被换出的大小             |
| RES      | 进程常驻内存，未被换出的物理内存,RES=CODE+DATA |
| CODE     | 可执行代码占用的物理内存大小                   |
| DATA     | 进程使用的数据段+栈占用的内存大小              |
| SHR      | 共享内存大小                                   |

## 进程

### systemd

使用systemd对进程进行管理，创建```dra.service```文件，并拷贝到`/usr/lib/systemd/system/`目录即可。

```shell
[Unit]
Description=HuaweiCloud Hybrid Disaster Recovery Appliance
After=network-online.target

[Service]
Type=forking
ExecStart=/opt/dra/script/dra.init start
ExecStop=/opt/dra/script/dra.init stop
PIDFile=/var/run/dra.pid
KillMode=process
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**配置开机自启动：**`systemctl enable dra`

**启动/停止/状态服务：**`systemctl start/stop/status dra`

其他详细使用可参考[这里](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)

### kill

| 信号    | 数字 | 描述                                                         |
| ------- | ---- | ------------------------------------------------------------ |
| SIGHUP  | 1    | 挂起，用来在终端连接丢失的时候通报                           |
| SIGINT  | 2    | 中断，在用户点击`Ctrl+C`时发送                               |
| SIGABRT | 6    | 通过C函数`abort`发送，为`assert`使用                         |
| SIGKILL | 9    | 迅速完全终止进程；不能被捕获这个强大和危险的命令迫使进程在运行时突然终止，进程在结束后不能自我清理。 |
| SIGTERM | 15   | 常规地终止进程                                               |
| SIGCONT | 18   | 在SIGSTOP后继续运行，类似fg/bg命令                           |
| SIGSTOP | 19   | 中断进程，不能被捕获。同`Ctrl+Z`                             |

### ulimit

### 其他

**删除所有匹配到name的进程：**`ps aux|grep name|awk '{print $2}'|xargs kill -9`
