---
title: "IO"
categories: ["python"]
tags: [""]
date: 2020-07-11T15:26:21+08:00
---

# IO

## 文件

文件打开方式列表：

| 代码 | 方式     |
| ---- | -------- |
| r    | 只读     |
| r+   | 读写     |
| a    | 附加写   |
| a+   | 附加读写 |
| w    | 新建只写 |
| w+   | 新建读写 |
| rb   | 二进制读 |

### 常用用法

**获取文件目录：** `os.path.basename(os.path.dirname(os.path.abspath(filename)))`

**按行读取文件： **

```python
with open(finename, 'r') as f:
  for line in f:
    do_something(line)
```

