---
title: "元编程"
categories: ["python"]
tags: ["metaclass"]
date: 2020-12-07T23:20:00+08:00
---

# 属性

## 属性描述符



## 属性查找顺序

实例代码：

```python
class MyAttr():
    name = 'class_levy'
    def __init__(self):
        self.name = "instance_levy"
    def __getattribute__(self, item):
        return "This is getattribute"
    def __getattr__(self, item):
        return "This is getattr"

my = MyAttr()
print(my.name)
```

# 参考

- [属性描述符官方文档](https://docs.python.org/3/howto/descriptor.html)

