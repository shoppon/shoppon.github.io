# 高级数据结构

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

