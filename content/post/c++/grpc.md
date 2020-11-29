---
title: "grpc学习"
date: 2018-09-14T23:58:34+08:00
categories: ["c++"]
draft: true
---

# grpc学习

## 学习目标

- 单元测试
- 线程调试
- Http2
- IO多路复用
  - Channel
- 跨平台实现
- 序列化与反序列化
- 代码生成
- 高级语法

## 架构

## 实现

### 语言特性

### 编码风格

#### 命名长度

个人一直以来比较喜欢简洁的命名，一般不是超过4个单词。在`grpc`中看到一个长达74个字符的名称`grpc_dns_lookup_ares_continue_after_check_localhost_and_ip_literals_locked`，在80个字符长度限制下几乎是理论最长。名称直接表示了流程，这样长的名称是否合理？

#### 命名类型

成员变量以`_`结尾，而非使用`m_xxx`的格式。

## 单元测试

### 编译

执行`bazel test //test/core/iomgr:tcp_server_posix_test --compilation_mode=dbg`编译

### 调试

