---
title: "高级数据结构"
categories: ["python"]
tags: ["数据结构"]
date: 2020-07-11T15:24:58+08:00
---

### defaultdict

含默认值的字典

```python
from collections import defaultdict

dd = defaultdict(list)
print(dd.get('foo')) # None
print(dd['foo]) # []
```

### Counter

统计一个iterable数据结构中元素出现的次数。

```python
from collections import Counter

c0 = Counter(['foo', 'bar'])
c1 = Counter('foobar')
```

### dequeue

双端队列

### namedtuple

使用'.'访问属性，一般用于模拟对象

```pyt
from collections import namedtuple

Foo = namedtuple('Foo', ['x', 'y'])
foo = Foo(x=1, y=2)
print(foo.x)
```

### heapq

```python
import heapq

q = heapq.heapify([1, 2, 3, 4, 5])
heapq.heappop(q)
```

### Array
相当于C语言数组，只能存放制定类型的属性

```python
arr = array.array(‘i’)
arr.append(‘abc’)
```

### bisect
维护已排序的序列（升序）

```python
foo = []
bisect.insert(foo, 2)
bisect.insert(foo, 5)
bisect.insert(foo, 3)
bisect.
```