---
title: "C/C++基础"
tags: ["c++"]
categories: ["c++"]
date: 2020-07-17T19:26:38+08:00
---

## 左值和右值

**左值：**可以寻址的变量或表达式，既可以出现在等号左边也可以出现在等号右边。

**右值：**右值只能被const类型的reference所指向，非const引用是非法的。

## 智能指针

#### 智能指针引用计数。。。转值

智能指针的引用是否导致计数变化？

#### 智能指针的的引用计数如何变化？从queue中移除是否导致变化？

#### 智能指针在引用计数变为0后内存是否立即回收？

## 类型转换

### 智能指针向下转型

使用`static_pointer_cast`进行向下转型

```c++
struct A {};
struct B: A {};

shared_ptr<A> foo = std::make_shared<A>();
shared_ptr<B> bar = std::static_pointer_cast<B>(foo);
```

参考[这里](http://www.cplusplus.com/reference/memory/static_pointer_cast/)

## 性能

对象拷贝的代价有多大？

在栈上分配然后返回和在堆上分配然后返回智能指针的区别？

## 宏

### 为什么宏都用`do while(0)`语法？

避免受到大括号、分号的影响，引起悬挂else等问题，具体见[这里](https://stackoverflow.com/questions/154136/why-use-apparently-meaningless-do-while-and-if-else-statements-in-macros)

## 工具

### pclint屏蔽

| 命令格式            | 说明                       | 举例                            |
| ------------------- | -------------------------- | ------------------------------- |
| -e#                 | 隐藏某类错误               | /*lint -e725 */                 |
| -e(#)               | 隐藏下一表达式中的某类错误 | /*lint –e(534) */               |
| !e#                 | 隐藏本行中的错误           | /*lint !e534 */                 |
| -esym(#, Symbol)    | 隐藏有关某符号的错误       | /*lint –esym(534, printf)*/     |
| -elib(#)            | 隐藏头文件中的某类错误     | /*lint –elib(129) */            |
| -efunc(#, \<func\>) | 隐藏某个函数中的特定错误   | /*lint –efunc(534, mchRelAll)*/ |

