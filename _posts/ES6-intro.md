---
title: "ES6总结笔记"
date: 2017-03-23 20:58:00
cover: "7.jpg"
category: "JavaScript"
tags:
    - JavaScript
---
>之前看了一些文章对ES6进行了介绍，现在系统地过一遍ES6入门，整理一下前后的心得体会。

# ES6总结笔记
<!--more-->

## `let`和`const`
* 块级作用域内有效
* 不会变量提升
  * 注意：使用Babel生成的ES5就可能会提升。但是这并不代表ES6中也会
  * “暂时性死区”就是在声明前使用变量会报错
  * **为什么不会变量提升？**：减少运行时错误，防止未声明先使用
* `let`不允许重复声明
* ES6存在块级作用域了，IIFE的作用减弱了
* 块级作用域中允许声明函数，且`function`关键字作用类似`let`，只在该块级作用域内有效；声明需谨慎，不同环境行为差异比较大
* `const`是常量，声明时必须初始化
  * 基本数据类型：值只读
  * 引用数据类型：指针只读，即指向固定的一个地方，至于这个地方是什么不一定
* BOM的`window`对象和Node的`global`对象将分离
* `let`、`const`、`class`声明的全局变量将不再是全局对象的属性；`var`、`function`不变

## 变量解构
* 按照一定方式对复杂数据结构中的内容进行parse.
  {% codeblock lang:javascript %}
  let [a, b, c] = [1, 2, 3];
  {% endcodeblock %}
* 解构不成功就`undefined`
* 等号右侧的数据结构应该是可遍历的
* 默认值
  {% codeblock lang:javascript %}
  let [foo = true] =[];
  {% endcodeblock %}
* 对象解构：对象属性没有“序”的概念，变量和属性同名才能取到正确的值

未完待续......
