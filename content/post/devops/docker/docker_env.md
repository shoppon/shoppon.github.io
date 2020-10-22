---
title: "使用docker构建开发环境"
categories: ["docker"]
tags: [""]
date: 2020-02-28T17:05:23+08:00
---

# 使用docker构建开发环境

## Docker的优势

软件开发必然要用到类似```gcc```、```gdb```、```cmake```等各种工作，为了避免对电脑运行环境的污染，最好使用```docker```搭建开发环境。

## Dockerfile

使用dockerfile实现零二进制构建开发环境

```shell
FROM ubuntu:18.04

COPY sources.list /etc/apt/sources.list

RUN apt update
RUN apt install -f -y gcc g++ cmake gdb perl vim git net-tools clang-format
RUN apt install -f -y openssh-server

RUN mkdir /run/sshd
RUN echo "root:ubuntu"|chpasswd

COPY id_rsa.pub /root
RUN mkdir -p /root/.ssh/ && cat /root/id_rsa.pub >> /root/.ssh/authorized_keys

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]

```

