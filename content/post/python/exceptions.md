---
title: "异常"
categories: ["python"]
tags: [""]
date: 2020-07-11T15:25:56+08:00
---

# 异常

## 用法

有`finally`语句存在时，抛出的异常会被忽略：

```python
def calc(division):
    try:
        return 1/division
    except ZeroDivisionError:
        raise ValueError("Invalid.")
    finally:
        return 0


print(calc(0)) # 0

```

捕获多个异常时使用**元组**：

```python
try:
    l = ['a', 'b']
    int(l[2])
except (ValueError, IndexError):
    pass
```

