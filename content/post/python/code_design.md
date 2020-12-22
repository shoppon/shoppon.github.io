---
title: "代码设计"
categories: ["python"]
tags: ["design"]
date: 2020-12-09T20:01:00+08:00
---

## 内聚的理解

在函数内判断条件返回，还是判断条件不调用函数。

```python
def foo():
  if not cond:
    return
  do_something()
  
def bar():
  foo()
  
# ------------------

def foo():
  do_something()
  
def bar():
  if cond:
    foo()
```

对`cond`的判断是放在`bar`函数内较好还是放在`foo`函数内较好。

