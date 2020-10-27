---
title: "数据结构"
date: 2020-07-17T19:26:38+08:00
categories: ["编程能力"]
tags: ["c++"]
draft: true
---

# STL

## List & Vector

Vector:

- 内存连续
- 预分配内存
- 每个元素仅需要其自身大小的空间（针额外指针空间）
- 当新添加元素时可自动重新分配内存
- 在尾部插入常数时间，其他位置O(n)时间。
- 在尾部删除常数时间，其他位置O(n)时间。
- 可以随机访问元素。
- 迭代过程中添加和删除元素是非法的。
- 可以获取底层数组。

List：

- 内存不连续
- 无预分配内存，单个元素所需内存大于其自身空间（前后指针)。
- 添加元素时不会自动重新分配内存。
- 无论在任何位置添加和删除的代价更小。
- 组合和分割列表代价更小。
- 无法随机访问元素，获取指定元素的代价更大。
- 迭代过程中可以添加和删除元素。
- 如果需要一个元素数组，必须创建一个然后添加元素。

# 结构体

### 字节对齐

使用`__attribute__`属性指定结构体对齐方式。

```c++
typedef struct
{
    char d1;
    int d2;
    unsigned short d3;
    char d4;
} Foo;
```

`sizeof(Foo)`为12。

```c++
typedef struct
{
    char d1;
    int d2;
    unsigned short d3;
    char d4;
} __attribute__((packed)) Bar;
```

使用`packed`指示编译器取消字节对齐，`sizeof(Bar)`为8。

### 对齐规则

- 从结构体首地址开始依次将元素放入内存，元素会被放置在其**自身对齐大小的整除倍数**地址上。`char`类型可以在任何地方开始，`short`类型开始地址必须被2整除，4位的`int`或者`float`类型开始地址必须被4整除，8位的`long`类型开始地址必须被8整除。
- 如果结构体大小不是所有元素中**最大对齐大小的整数倍**，则结构体对齐到最大元素对齐大小的整数倍，填充空间放置到结构体末尾。
- 结构体数据类型的对齐大小为其自身元素中最大对齐大小元素的对齐大小。

## 参考

- [What is the meaning of “__attribute__((packed, aligned(4))) ”](https://stackoverflow.com/questions/11770451/what-is-the-meaning-of-attribute-packed-aligned4)

- [structure-packing](http://www.catb.org/esr/structure-packing/)

- [C/C++中结构体字节对齐规则](https://zhuanlan.zhihu.com/p/26122273)
