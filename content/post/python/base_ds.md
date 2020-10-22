---
title: "基础数据结构"
categories: ["python"]
tags: [""]
date: 2020-06-11T21:26:38+08:00
---

# 基础数据结构

## 数值

python共有4种数字类型`int`、`long`、`float`、`complex`。

`quatize`方法将数字舍入为固定指定。

```python
from decimal import *
print(Decimal('7.325').quantize(Decimal('.01'), rounding=ROUND_DOWN)) # 7.32
print(Decimal('7.325').quantize(Decimal('1.'), rounding=ROUND_UP)) # 8
```

`bool`是`int`的子类，可以与整形比较

```python
>>> bool('0')
True
>>> bool('1')
True
>>> bool(0)
False
>>> bool(0.1)
True
>>> 1.0 > True
False
>>> 0.1 < True
True
>>> 1.1 > True
True
>>> -0.1 < False
True
```

## 集合

集合的交集、并集、差集、对称差集（并集减去交集）。

```python
>>> foo = {1, 2, 3, 4}
>>> bar = {4, 5, 6}
>>> print(foo ^ bar)  # 对称差集
{1, 2, 3, 5, 6}
>>> print(foo & bar)  # 交集
{4}
>>> print(foo | bar)  # 并集
{1, 2, 3, 4, 5, 6}
>>> print(foo - bar)  # 差集
{1, 2, 3}
>>> print(foo.symmetric_difference(bar))
{1, 2, 3, 5, 6}
```

集合的`unpack`与原始顺序并不一致。

```python
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> a, b, c={'a', 'b', 'c'}
>>> print(a, b, c)
b c a
>>> 

[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
>>> a, b, c={'a', 'b', 'c'}
>>> print(a, b, c)
a c b
>>> 
```

## 字符串

### 切片

**指定切片间隔：**

```python
s = 'foofoofoo'
print(s[::3]) # 'fff'
```

**范围为前闭后开：**

```python
s = 'foo'
print(s[1:2]) # 'o'
arr = [1, 2, 3]
print(arr[:-1]) # [1, 2]
print(arr[:]) # [1, 2, 3]
```

**给切片赋值**

```python
l = list(range(5))
l[2:4] = [8]
print(l) # [0, 1, 8, 4]
```

### 格式化

`format`函数参数可用`{x}`来获取，`f`为浮点格式，`d`为整数格式。

```python
>>> '{2}-{1}-{0}'.format(1 ,2 ,3)
'3-2-1'
>>> '{0[0]}-{0[1]}-{1[1]}'.format([1, 2, 3], ['x', 'y', 'z'])
'1-2-y'
>>> '{0}'.format([1, 2, 3])
'[1, 2, 3]'
>>> '{0:1d}'.format(1)
'1'
>>> '{0:3d}'.format(1)
'  1'
>>> '{0:02d}'.format(1)
'01'

>>> f'{123.456:08.2f}'
'00123.46'
>>> f'{123.456:8.2f}'
'  123.46'
```

## 列表

列表比较按字典序排序

```python
>>> ['a', 'b', 'c'] < ['a', 'c', 'b']
True
>>> [1, 2] < [3]
True
```

列表遍历时删除

```python
>>> arr = list(range(5))
>>> for i in arr:
...   arr.remove(i)
... 
>>> arr
[1, 3]
```

## 运算符

多个比较运算符同时存在是遵循**链式比较**。

```python
>>> 1 < 2 < 3 
True # (1 < 2) and (2 < 2)
```

使用双除号向下取整：`5 // 2`
