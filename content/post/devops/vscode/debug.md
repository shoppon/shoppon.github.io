---
title: "VSCode调试手册"
categories: ["vscode"]
tags: ["vscode", "debug"]
date: 2020-12-07T16:00:00+08:00
---

# 调试配置

## 调试所有代码

VSCode默认只调试工程中的代码，如果需要调试外部代码需要手动开启。

在`lanuch.json`不添加以下配置，关闭`justMyCode`。

```javascript
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "unittest",
            "type": "python",
            "request": "test",
            "justMyCode": false,
            "console": "integratedTerminal"
        }
    ]
}
```

