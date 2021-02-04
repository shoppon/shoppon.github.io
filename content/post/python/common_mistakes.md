---
title: "python常见错误"
categories: ["python"]
tags: ["python"]
date: 2020-12-25T14:30:00+08:00
---

# 编码规范

## 单元测试

严格上讲，所有业务代码都必须有单元测试防护。单元测试是效率最高的验证手段。

## 日志记录

对于所有非查询类接口，应当在入口处打印日志，并记录详细的参数，在返回处也应该打印，记录返回值。

# 异常处理

## 边界值

一般而言，外界的输入都要认为是不可信。函数需要对输入进行各种检查，尤其要考虑各种边界值。

字符串要考虑空串、特殊字符。

数字类型要考虑0、负值等。

> ```python
> def foo(some_str):
>   	items = some_str.split('')
>     if items[1] == 'bar':
>        do_something()
> ```
>
> 在该案例中，必须对空串的情况进行校验。


## 特殊值

## 堆栈打印

堆栈对问题定位至关重要，在出异常时应该打印堆栈。程序员有堆栈一般一秒就能想到问题所在，如果没有堆栈需要结合上下文分析原因。

> ```python
> def foo():
>   	try:
>     		do_something()
>   	except Exception:
>     		LOG.err("An exception occurred.")
> ```
>
> 这里应该用`LOG.exception`打印堆栈。

## 异常抛出

捕获异常然后又重新抛出时，最好只使用`raise`，这样会保持异常的堆栈，如果抛出新的异常，会导致原始出异常的地方堆栈丢失。

> ```python
> def foo():
>   try:
>     do_something()
>   except Exception:
>   	raise NewException  
> ```
>
> 这里应该用`raise`直接抛出

# 性能问题

## 数据库性能

### 避免多次单个操作

应当尽量减少操作数据库的次数，尽量提升数据库语句的效率。

## 迭代器性能

如果需要从一个列表中获取第一个满足条件的元素，应该使用`next`函数，而不是使用`filter`获取所有再取第一个。

> ```python
> def get_first_match(arr):
>   items = filter(lambda x: x.name == 'foo', arr) # bad
>   first = next(x for x in arr if x.name == 'foo') # good
> ```
>
> 如果不想抛出`StopIteration`异常，可添加默认值。[参考](https://stackoverflow.com/questions/2361426/get-the-first-item-from-an-iterable-that-matches-a-condition)

