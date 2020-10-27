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

# 参考

- [从源码安装borg](https://borgbackup.readthedocs.io/en/stable/installation.html#using-git)

