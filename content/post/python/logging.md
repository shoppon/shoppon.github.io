---
title: "日志"
categories: ["python"]
tags: [""]
date: 2020-07-11T15:26:50+08:00
---

# 日志

## logging

默认将`WARNNING`级别以上打印到`std.out`：

```python
>>> logging.info('info')
>>> logging.warning('warn')
WARNING:root:warn
>>> logging.warning('error')
WARNING:root:error
>>> 
```

## 日志格式

