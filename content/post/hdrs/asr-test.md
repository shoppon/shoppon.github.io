---
title: "ASR分析"
categories: ["hdrs"]
tags: [""]
date: 2020-02-28T17:53:42+08:00
---

## 特性分析

### 重新同步

## 测试用例

### 盘恢复后是否重同步

**测试步骤**

1. 在主机上写入文件/home/x0025023/mock.dat
a. dd if=/dev/urandom of=/home/x00250203/mock.dat bs=1024 count=4
b. stat /home/x00250203/mock.dat---15865596
c. debugfs -R "stat <19>" /dev/sda
d. echo "hello world" > /home/x00250203/text.dat
2. 对主机进行备份
3. 修改mock.dat内容为hello asr
a. dd if=/dev/urandom of=/home/x00250203/mock.dat bs=1024 count=4---15814536
b. echo "hello asr" > /home/x00250203/text.dat
4. 待文件同步到asr
5. 关机，使用主机备份进行恢复
a. 文件变为hello world
6. 等待一段时间，所有数据均同步到asr
7. 在ASR上进行failover，查看文件内容

**预期目标（有一致性校验）：**

	1. /home/x00250203/text.dat文件内容为hello world

**预期目标（无一致性校验）：**

	1. /home/x00250203/text.dat文件内容为hello asr

![image-20190928230045785](/Users/shoppon/Library/Application Support/typora-user-images/image-20190928230045785.png)