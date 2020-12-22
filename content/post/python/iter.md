---
title: "迭代器"
categories: ["python"]
tags: [""]
date: 2020-07-11T15:26:35+08:00
---

# 迭代器

迭代器可通过`next`函数获取下一个值：

```python
>>> iter
<built-in function iter>
>>> i  = iter('abc')
>>> type(i)
<class 'str_iterator'>
>>> next(i)
'a'
```

## 自定义迭代器

自定义迭代器时`__iter_`函数返回自身`self`，`__next__`返回值，全部返回时`raise StopIteration`。

```python
class MyRange:
    def __init__(self, start, end):
        self.start = start
        self.end = end

    def __iter__(self):
        return self

    def __next__(self):
        if self.start < self.end:
            self.start += 1
            return self.start-1
        else:
            raise StopIteration


for i in MyRange(1, 3):
    print(i)  # 1 2
for i in range(1, 3):
    print(i)  # 1 2
```

## 生成器

使用`yield`定义生成器

```python
def foo():
  yield 0

type(foo) # <class 'function'>
type(foo()) # <class 'generator'>
```

## 推导式

### 列表推导式

### 生成器表达式
```python
foo = (i for i in range(10) if i % 2 == 0)
print(type(foo))
```

### 字典推导式

### 集合推导式