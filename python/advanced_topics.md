# Python进阶主题

## 高级数据结构

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

## 协程

## 字符串编码

## 上下文管理器

## 元类

## 并发与同步

## 单元测试

## 魔法函数

### __repr__

### map

将iterable生成新map的对象

```python
res = map(lambda x, y: x + y, [1, 3, 5, 7, 9], [2, 4, 6, 8, 10])
print(list(res))
```

### zip

同步遍历多个iterable对象

```python
for x, y in zip([1, 3, 5], [2, 5, 6]):
    print(x)
```

## 偏函数

## 运算

使用双除号向下取整：`5 // 2`
