---
title: Python学习笔记(一)-关于基本语法和异常值类型
date: 2017-01-19 20:00:00
cover: '6.jpg'
category: "Python"
tags:
    - Python
---

## 基础入门
目前我最喜欢Python的一点是它具有一些非常方便易用的数据结构。
* `list[]`
  * list是一个有序集合，类似其他语言中的数组
  * list中元素可以增删，可以追加元素到末尾
* `tuple()`
  * tuple也是有序集合
  * __初始化之后不可增删改__
<!-- more -->
* `dict`
  * 类似其他语言的Hash-map，以key-value形式存储数据
  * 随机访问速度很快，通过key能够迅速找到value
  * 插入速度极快
  * 空间换时间的数据结构
* `set`
  * 可以看成是key的集合，不存储value
  * 特点类似数学意义上的集合：
    * 无序
    * 不重复
    * 能够进行集合运算（交并补）
* 可变与不可变对象
  * 不可变：初始化后其值/其元素值不改变：str、tuple等
  * 可变对象：初始化后随着程序运行值可能发生改变、增删的，如list
* Python的map-reduce
  * `map(f, [1, 2, 3])`将传入函数的`f`映射应用于list中的每一个值
  * `reduce(g, [1, 2, 3])`中传入函数`g`一般是二元函数，它将作用于list的前两个值上，得到结果后，下一步将函数作用于该结果和下一个值

## 常见异常类型
1. `NameError`：尝试访问一个未申明的变量
```
>>>  v
NameError: name 'v' is not defined
```
2. `ZeroDivisionError`：除数为0
```
>>> v = 1/0
ZeroDivisionError: int division or modulo by zero
```

3. `SyntaxError`：语法错误
```
>>> int int
SyntaxError: invalid syntax (<pyshell#14>, line 1)
```

4. `IndexError`：索引超出范围
```
>>> List = [2]
>>> List[3]
Traceback (most recent call last):
  File "<pyshell#18>", line 1, in <module>
    List[3]
IndexError: list index out of range
```

5. `KeyError`：字典关键字不存在
```
>>> Dic = {'1':'yes', '2':'no'}
>>> Dic['3']
Traceback (most recent call last):
  File "<pyshell#20>", line 1, in <module>
    Dic['3']
KeyError: '3'
```

6. `IOError`：输入输出错误
```
>>> f = open('abc')
IOError: [Errno 2] No such file or directory: 'abc'
```

7. `AttributeError`：访问未知对象属性
```
>>> class Worker:
 def Work():
  print("I am working")
>>> w = Worker()
>>> w.a
Traceback (most recent call last):
  File "<pyshell#51>", line 1, in <module>
    w.a
AttributeError: 'Worker' object has no attribute 'a'
```

8. `ValueError`：数值错误
```
>>> int('d')
Traceback (most recent call last):
  File "<pyshell#54>", line 1, in <module>
    int('d')
ValueError: invalid literal for int() with base 10: 'd'
```

9. `TypeError`：类型错误
```
>>> iStr = '22'
>>> iVal = 22
>>> obj = iStr + iVal;
Traceback (most recent call last):
  File "<pyshell#68>", line 1, in <module>
    obj = iStr + iVal;
TypeError: Can't convert 'int' object to str implicitly
```

10. `AssertionError`：断言错误
```
>>> assert 1 != 1
Traceback (most recent call last):
  File "<pyshell#70>", line 1, in <module>
    assert 1 != 1
AssertionError
```
