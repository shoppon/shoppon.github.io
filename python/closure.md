# 闭包及装饰器

## 概念

**闭包**是一种函数，它会保留定义函数时存在的**自由变量**的绑定，这样调用函数时，虽然定义**作用域不可用**了，但是的**仍能**使用那些绑定。闭包**延长**了非全局变量的作用域，体现了**封装性**。

```python
def make_averager():
    seriers = []

    def averager(new_value):
        seriers.append(new_value)
        return sum(seriers) / len(seriers)
    return averager

avg = make_averager()
print(avg(10))
print(avg(15))
print(avg(17))
print(avg.__code__.co_varnames)
print(avg.__code__.co_freevars)
print(avg.__closure__[0].cell_contents)
```

`seriers`是闭包`make_averager`的自由变量。

### 作用域范围

python不要求声明变量，但是假定在函数体中赋值的变量是局部变量

```python
var = 1
def foo(bar):
    print(bar)
    print(var) # UnboundLocalError: local variable 'var' referenced before assignment
    var = 2
foo(5)
```

如果去掉`var = 2`这行则不会报错。

## 装饰器

### 装饰器

带参数的装饰器就是在不带装饰器的基础上再包装一层：

```python
import time
import functools

DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

def clock(fmt=DEFAULT_FMT):
    def decorate(func):
        @functools.wraps(func)
        def clocked(*_args):
            t0 = time.time()
            _result = func(*_args)
            elapsed = time.time() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)
            result = repr(_result)
            print(fmt.format(**locals()))
            return _result
        return clocked
    return decorate

@clock(fmt='{name}:{elapsed}:outter')
@clock(DEFAULT_FMT+':innner')
def foo(seconds):
    time.sleep(.1)

foo(1)
```

`clock`是参数化装饰器的工厂函数，`decorate`是真正的装饰器，`clocked`包装被装饰的函数。

### 多重装饰器

多个装饰器叠加使用时，**最近的最先执行**：

```python
@clock(fmt='{name}:{elapsed}:outter')
@clock(DEFAULT_FMT+':innner')
def foo(seconds):
    time.sleep(.1)

foo(1) 
# [0.10014772s] foo(1) -> None:innner
# foo:0.10019731521606445:outter
```

### 内置装饰器

用于函数内部的三个装饰器`classmethod`、`staticmethod`和`property`。

使用`lrc_cache`实现备份功能，将耗时的函数结果保存起来

```python
@functools.lru_cache()
@clock()
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-2) + fibonacci(n-1)

fibonacci(5)
# [0.00000048s] fibonacci(1) -> 1
# [0.00000024s] fibonacci(0) -> 0
# [0.00000691s] fibonacci(2) -> 1
# [0.00002027s] fibonacci(3) -> 2
# [0.00000048s] fibonacci(4) -> 3
# [0.00003028s] fibonacci(5) -> 5
```

每个数字的`fibonacci`只计算了一次。
