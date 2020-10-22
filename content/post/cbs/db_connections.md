---
title: "数据库连接数问题"
categories: ["cbs"]
tags: [""]
date: 2020-04-17T19:39:16+08:00
---

# 数据库连接数问题

### 问题现象

6000虚拟机并发备份时，大量备份处于waiting_protect状态。

日志中大量打印async_connection_failed。

### 攻关过程

为什么连接失败，备份时连接数快速达到2000。查看数据库配置发现连接数正好是2000。

写程序模拟12进程，并发1800访问数据库查询备份，未发现超时。

连接池设置的是10，max_overflow是100，4x3x2个进程的情况下。

以为是数据库连接配置不对，将连接池改成size 10，max_overflow 20，结果还是超了。

怀疑taskflow占用太多，使用karbor_b账号给taskflow，结果taskflow永远只有40，查询taskflow连接db的方式engine.get_conn。

怀疑with session.begin的方式不会把连接归还给连接池。

本地调试发现session begin的时候会起transaction，close_with_result会被重置为False。

通过LLT调试发现确实没有调用QueuePool的return_conn，走了弯路，后来发现sqlite用的是StaticPool。

参考Nova、Cinder代码，发现没有一处使用session.begin的方式。

将所有session.begin去掉，再次调试，发现还是超出。

怀疑Sqlalchemy的pool实现有问题，overflow控制不住，会一起超，添加日志。

单个备份调试，发现连接池开始是正常的，一直正常的get、return，突然又create，苦思不得其解，怀疑queue的实现有问题。找际荣一起看，一语惊醒梦中人，有可能连接池被重置了。

本地调试，在连接池初始化的地方打断点，然后备份，看哪些地方会重置连接池，checker、controller、rpc_api等地方基类会dispose_engine。

### 优化建议

并发调度时时间打散，让任务均匀分布在轮询间隔内，最简单可用随机数实现。

精简flow创建流程，当前创建时保存了resources，parameters，在CBR里面不需要。（历史原因，无策略手工备份）

### 改进措施

观念转变，每一行代码都认真思考是否有提升空间，比如db查询是否可以减少，循环次数是否能减少，是否可以用缓存，一个不经心的代码可能导致全局的性能瓶颈。

引入先进的工具评估代码的性能。有些代码表面上看起来很简单，实际上可能蕴含了巨大的问题，而且通过代码检视可能发现不了，比如rpc_api初始化时基类会重置连接池，这是检视万万发现不了的。需要用工具来评估代码的性能，类似cProfile、jmeter、mockserver这样的工具。
