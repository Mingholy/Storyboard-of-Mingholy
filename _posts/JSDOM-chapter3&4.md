---
title: "JavaScript DOM 编程艺术 笔记-第三章 第四章"
date: 2017-02-16 20:21:00
cover: '5.jpg'
category: "JavaScript"
tags:
    - JavaScript
    - 前端
---
> 其实这些内容很早之前就遇见过，也实际应用过。但是当时脑子里根本没有一些基本的概念，只知道这样做有效就这样做了。后来渐渐懂了一些道理，但是深深的感觉到还是野路子出身。基础知识还是要扎实才好，巩固的工作还是要做的。这也是为什么回过头来认认真真地看这一章，总结出这些内容。

<!-- more -->

## 第三章：DOM
DOM：Document Object Model  
D是document毫无疑问，值得注意的是O和M。  
### O和M
#### O-Object-对象
JS里对象分为三类：
1. 用户定义对象
2. 内键对象：`Array`、`Math`、`Date`等，JavaScript自带的对象
3. 宿主对象：由浏览器提供的对象，如`window`

`window`对象对应浏览器窗口，由浏览器提供。它的属性和方法就是BOM：`window.innerHeight`, `window.open`, `window.blur`等。

#### M-Model-模型
即Model，其实说的就是HTML节点树的形态描述

### 一些原生DOM方法
1. `getElementById`
2. `getElementsByTagName`: 返回对象数组
3. `getElementsByClass`: H5新增，类选择器

注意，上述方法是`document`对象的方法。

### 获取和设置属性
这些方法就是普通对象`object`的方法了。
1. `object.getAttribute(attribute)`
2. `object.setAttribute(attribute, value)`

## 第四章：实例：图片库
这一章主要是实例，其中需要注意的几个点总结如下。
### 如何在元素上阻止`onclick()`的冒泡事件
首先，如果在`a`元素上绑定了`onclick()`这样的事件，在点击它的时候，`onclick`事件处理函数__调用的函数__默认返回值是`true`，对于该事件处理函数来说，它会认为自己受到了一次点击，就要转到`a`元素所指向的链接。  
反之如果该调用函数返回了`false`，那么事件处理函数将不会触发这个链接的默认行为，则不会跳转。所以这里的处理方式的，给调用函数的代码处添加`false`返回值。
{% codeblock lang:html %}
<a href="http://www.example.com" onclick="showPic(); return false;"></a>
{% endcodeblock %}

### 关于`node`的各种属性和方法
首先是子元素：
1. `childNodes`属性：返回一个子元素数组
2. `nodeType`属性，可以根据值区分节点类型：
  1. 值为1：元素节点
  2. 值为2：属性节点
  3. 值为3：文本节点
3. `nodeValue`属性：它可以取到一个**文本节点**的值；给它赋值也可以改变该文本节点的值。
4. `firstChild`和`lastChild`：见名知意。
