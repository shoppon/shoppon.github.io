---
title: "字符串"
categories: ["python"]
tags: [""]
date: 2020-06-11T21:28:03+08:00
---

# 字符串

## 概念

- unicode是**字符集**，utf-8/gbk的**编码规则**
- python2中str存储**字节码**(bytes)
- unicode使用编码转换成str， `u'中文'.encode('utf-8')`，在python3中已经废弃
- Python3中的str相当于python2中的unicode

**unicode字符集：**每一个符号对应一个编码，中文从U+4E00～U+9FFF。

**字节码：**字符串(unicode)经过**编码**形成字节码，以`b'`开头。

**字符串：**字节码经过**解码**形成字符串。

```python
>>> bytes_code = '中文'.encode()
>>> type(bytes_code)
<class 'bytes'>
>>> '中文'.encode()
b'\xe4\xb8\xad\xe6\x96\x87'
>>> '中文'.encode('utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'
>>> '中文'.encode('gbk')
b'\xd6\xd0\xce\xc4'
>>> '中文'.encode('utf-8').decode()
'中文'
```

### 字节码和字符串转换

字节码转换为字符串：

```python
bytes_code = b'foo'
str_type = str(bytes_code, encoding='utf-8')
```

字符串转字节码：

```python
str_type = 'foo'
bytes_code = bytes(str_type, encoding='utf-8')
```

### UTF-8编码规则

- 变长
- 单字节符号，字节每1位为0，可表示asci码
- 对于n字节的符号，每1个字节的前n位都设为1，后面的n+1位设为0，通过1的位数即可知道占用字节数。后面字节的前两位全部设置为10（why?），剩下没有设置的位则表示unicode码。

扩展：自定义实现Utf-8编码。

### 大端对齐与小端对齐

高位字节保存到高位地址，低位字节保存在低位地址称之为大端对齐（大端序），反之为小端对齐。

unicode规范中，如果一个文本文件的头两个字节是**FE FF**，就表示该文件采用**大头**方式；如果头两个字节是**FF FE**，就表示该文件采用**小头**方式。

### 转义

单引号和双引号无区别，混合使用时无需转义。

```python
>>> "a\"b"
'a"b'
>>> "a'b"
"a'b"
>>> """a'b"""
"a'b"
>>> 'a''b''c'
'abc'
>>> 'a"b'
'a"b'
```

### 查找

可通过`find`、`rfind`、`index`等多种方式查找：

```python
>>> 'abcab'.find('ab')
0
>>> 'abcab'.rfind('ab')
3
>>> 'abcab'.index('ab')
0
>>> 'abcab'.find('ac')
-1
>>> 'abcab'.rfind('ac')
-1
>>> 'abcab'.index('ac')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: substring not found
```

### 替换

python3中`replace`是全部替换：

```python
>>> 'This is a test'.replace('s', 'a')
'Thia ia a teat'
```

## 其他

**查看默认解码方式：**`sys.getdefaultencoding()`
