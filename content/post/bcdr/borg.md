```
title: "开源备份软件borg分析"
categories: ["灾备"]
tags: ["开源", "备份"]
date: 2020-10-26T20:02:00+00:00
```

# Borg分析

## 编译

执行`https://borgbackup.readthedocs.io/en/stable/installation.html#using-git`编译borg。

## 调试

vscode调试配置文件：

```shell
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "borg",
            "type": "python",
            "request": "launch",
            "module": "borg",
            "args": [
                "create",
                "/home/xp/borg_repo/my_repo::1026",
                "/home/xp/borg_test/fake.dat"
            ],
            "cwd": "${workspaceFolder}/src/"
        }
    ]
}
```

# 架构

## 数据结构

Index

Item

## 工程技术
使用`msgpack`进行序列化。

使用`fuse`将s3挂载成文件系统读写。不适合对已有文件经常修改，性能不如SDK。


# 流程

文件以`segment`形式进行存储，前八个字节是`MagicNumber`，内容为`b'BORG_SEG'`，用来判断格式是否正确，接着九个字节是`header`，标识是否已经`commit`。

```python
header_fmt = struct.Struct('<IIB')
assert header_fmt.size == 9
put_header_fmt = struct.Struct('<IIB32s')
assert put_header_fmt.size == 41
header_no_crc_fmt = struct.Struct('<IB')
assert header_no_crc_fmt.size == 5
crc_fmt = struct.Struct('<I')
assert crc_fmt.size == 4
```

使用`CRC`校验码判断内容是否被篡改`crc32(data, crc32(memoryview(header)[4:])) & 0xffffffff != crc:`

每个目录最多1000个文件，总共最多`2^32-1`个文件。

使用`LRUCache`缓存最近打开的`segment`句柄。

# 常用命令

查询repo信息：`borg info /home/xp/borg_repo/my_repo/`

列举备份详情：`borg info /home/xp/borg_repo/my_repo/::102`

列举备份列表：`borg list /home/xp/borg_repo/my_repo/`

列表备份文件列表：`borg list /home/xp/borg_repo/my_repo/::first`

解压备份：`borg extract /home/xp/borg_repo/my_repo/::102`

删除备份：`borg delete /home/xp/borg_repo/my_repo/::first`

# 参考

- [从源码安装borg](https://borgbackup.readthedocs.io/en/stable/installation.html#using-git)

