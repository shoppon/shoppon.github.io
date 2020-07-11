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

