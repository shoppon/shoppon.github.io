---
title: "正则表达式"
categories: ["python"]
tags: [""]
date: 2020-07-11T15:27:11+08:00
---

# 正则表达式

正则表达式一般默认是贪婪的，但是Python好像并不是如此：

```python
import re

ret = re.match(r'(.*)-(.*)-(.*)', 'abc-de-fg')
print(ret) # <re.Match object; span=(0, 9), match='abc-de-fg'>
g0 = ret.group 
print(g0) # <built-in method group of re.Match object at 0x7fc9fda9ce90>
print(g0(0), g0(1), g0(2), g0(3)) # abc-de-fg abc de fg
g1 = ret.groups 
print(g1) #<built-in method groups of re.Match object at 0x7fc9fda9ce90>
print(g1(0), g1(1), g1(2)) # ('abc', 'de', 'fg') ('abc', 'de', 'fg') ('abc', 'de', 'fg')

ret1 = re.match(r'(.*)-(.*)', 'abc-de-fg')
print(ret1) # <re.Match object; span=(0, 9), match='abc-de-fg'>
g3 = ret1.group
print(g3(0), g3(1), g3(2)) # abc-de-fg abc-de fg
```

`re.math`从头开始匹配，`re.search`可以从任意位置匹配：

```python
pattern = 'back'
print(re.match(pattern, 'backup.txt')) # <re.Match object; span=(0, 4), match='back'>
print(re.match(pattern, 'fake.back')) # None
print(re.search(pattern, 'backup.txt')) # <re.Match object; span=(0, 4), match='back'>
print(re.search(pattern, 'fake.back')) # <re.Match object; span=(5, 9), match='back'>
```

更多详细内容可参考[官方文档](https://docs.python.org/zh-cn/3/library/re.html)。

