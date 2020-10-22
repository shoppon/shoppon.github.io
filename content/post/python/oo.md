---
title: "面向对象"
categories: ["python"]
tags: [""]
date: 2020-06-11T21:27:38+08:00
---

# 面向对象

## 拷贝

**基本类型的拷贝均是深拷贝。**

浅拷贝会构造一个新的复合对象，然后将原对象中找到的**引用**插入其中。

深拷贝会构造一个新的复合对象，然后递归地将原始对象中所找到的**副本**插入。

`dict.copy()`、`list()`、切片`[:]`等都是浅拷贝。

```python
a = 1
b = a
b = 2
print(a) # 1

l1 = [1, 'a']
l2 = list(l1)
l2[0] = 'x'
print(l1) # [1, 'a']
print(l2) # ['x', 'a']

l3 = [[1, 'a'], [2, 'b']]
l4 = list(l3)
l4[1][0] = 'y'
print(l3) # [[1, 'a'], ['y', 'b']]
print(l4) # [[1, 'a'], ['y', 'b']]
```

## 继承

以**单下划线**开头的成员变量叫**保护变量**。

以**双下划线**开头的方法无法**多态**，不能被外部访问：

```python
class Foo(object):
    def f0(self):
        print('foo_f0')

    def _f1(self):
        print('foo_f1')

    def __f2(self):
        print('foo_f2')

    def call_f0(self):
        self.f0()

    def call_f1(self):
        self._f1()

    def call_f2(self):
        self.__f2()


class Bar(Foo):
    def f0(self):
        print('bar_f0')

    def _f1(self):
        print('bar_f1')

    def __f2(self):
        print('bar_f2')

    def __f3__(self):
        print('bar_f3')


bar = Bar()
bar.call_f0()  # bar_f0
bar.call_f1()  # bar_f1
bar.call_f2()  # foo_f2
bar.__f3__()  # bar_f3
bar.__f2()  # 'Bar' object has no attribute '__f2'
```

## 属性

### 类方法

类方法方法类变量时看**调用者**

```python
class Foo(object):
    instance = 0

    @classmethod
    def add(cls):
        cls.instance += 1

    def __init__(self):
        self.add()

class Bar(Foo):
    instance = 0

foo = Foo()
bar = Bar()
print(foo.instance, bar.instance) # 1 1
print(Foo.instance, Bar.instance) # 1 1
```

### 静态方法

静态方法访问类变量是看**变量所有者**：

```python
class Foo(object):
    instance = 0

    @staticmethod
    def add():
        Foo.instance += 1

    def __init__(self):
        self.add()

class Bar(Foo):
    instance = 0

foo = Foo()
bar = Bar()
print(foo.instance, bar.instance) # 2 0
print(Foo.instance, Bar.instance) # 2 0
```

