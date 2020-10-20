# 函数

## 参数

数组作为默认参数时，初始化时会默认生成一个数组：

```python
class Foo(object):
    def do(self, param=[]):
        param.sort()
        param.append('End')
        return param


print(Foo().do()) # ['End']
print(Foo().do()) # ['End', 'End']
print(Foo().do(param=["foo", "bar"])) # ['bar', 'foo', 'End']
print(Foo().do()) # ['End', 'End', 'End']
```

参数传递方式为**基本类型**是**值传递**，**非基本类**型是**引用传递**：

```python
def foo(some_number, some_list):
    some_number = 2
    some_list.append(2)

some_number = 1
some_list = [1]
foo(some_number, some_list)
print(some_number, some_list) # 1 [1, 2]
```

`for`循环的变量会当成局部变量，列表推导式不会：

```python
i = 3

def foo0(x):
    def bar():
        return i
    for i in x:
        pass
    return bar()

def foo1(x):
    def bar():
        return i
    [i for i in x]
    return bar()

print('%s,%s' % (foo0(['x', 'y']), foo1(['x', 'y']))) # y,3
```

### 可变参数

函数可变参数有两种：元组和字典

```python
def func(*args, **kwargs):
    print(args, kwargs, *args, **kwargs, sep='#')

func((1, 2), {'a': 0, 'b': 1}) # ((1, 2), {'a': 0, 'b': 1})#{}#(1, 2)#{'a': 0, 'b': 1}
func(1, 2, {'a': 0, 'b': 1}) # (1, 2, {'a': 0, 'b': 1})#{}#1#2#{'a': 0, 'b': 1}
func({'a': 0, 'b': 1}) # ({'a': 0, 'b': 1},)#{}#{'a': 0, 'b': 1}
func(1, 2, a=1, b=2) # (1, 2)#{'a': 1, 'b': 2} TypeError: 'a' is an invalid keyword argument for print()
```

具体规则如下：

- 元组参数一定要放在字典参数之前。
- 可变参数必须放在普通参数之后。
- 可变参数在函数调用时必须**解封后传递**，否则会当成一个函数处理。

### 作用域

局域变量并不会覆盖全局变量：

```python
def f():
    def innner():
        nonlocal n
        n = 2
    n = 1
    print(f'befor inner: {n}')
    innner()
    print(f'after innner: {n}')


f()
print(f'after f: {n}')

# befor inner: 1
# after innner: 2
# after f: 0
```

## 偏函数



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

