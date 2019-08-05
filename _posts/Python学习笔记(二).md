---
title: Python学习笔记(二)-关于切片、迭代器和生成器
date: 2017-01-22 20:00
cover: '6.jpg'
category: "Python"
tags:
    - Python
---
## 切片(Slice)
### 基本方法
Python中对List和Tuple可以进行切片操作，很方便地取出列表或元组中位于特定位置的元素。

基本使用方式是：
```
L[start_index:end_index]
```
我猜其中`start_index`默认为0，即List的起始元素位置；`end_index`默认为-1，即List的最后元素位置。
如果不标明两个参数，则按照默认值对待。
<!-- more -->
### Extended Slice
这种使用方法将带有第三个参数：
```
L[start_index:end_index:step]
```
其中step是取元素的步长。

## 迭代与迭代器
### 迭代与可迭代对象
__迭代(`Iteration`)__：是一种操作，最简单的是与`for`有关的，比如`for item in list:`这样的简单循环。  
__可迭代对象(`Iterable`)__：可以像上面那用使用`for`进行迭代的都叫做可迭代对象，比如`list`、`tuple`、`dict`、`set`、`str`。  
可迭代对象的判断：可以用`isinstance(obj, Iterable)`来判断是否是`Iterable`

### 迭代器
__迭代器(`Iterator`)__可以使用`next()`不断调用并返回下一个值。
迭代器的判断：
1. 可以被`next()`函数调用
2. 可以使用`isinstance(obj, Iterator)`

>`Iterable`包括：`list`、`dict`、`str`、`set`、`tuple`
    `Iterator`包括各种生成器。

## 生成器
### 列表生成器
给定生成一个序列的方法，根据需要来计算后续元素。这种__边循环边计算__的机制，叫做__生成器(`Generator`)__。
{% codeblock lang:python %}
L = [x * x for x in range(10)]  #list
G = (x * x for x in range(10))  #generator
{% endcodeblock%}
### 函数生成器
当函数的返回值是一串可由特定方法计算的值时，可以使该函数成为生成器。
{% codeblock lang:python %}
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
{% endcodeblock %}

其中`yeild`关键字使得该函数成为一个生成器。`yield`关键字的作用是，执行到该句时，保存调用现场，然后返回后面的值`b`。在使用`next()`再次调用该函数时，从`yield`__之后__继续执行。
