---
title: "JavaScript DOM 编程艺术 笔记-第六章"
date: 2017-03-17 14:00:00
cover: "1.jpg"
category: "JavaScript"
tags:
    - JavaScript
---
## 优化一个已有项目的思路
这一章主要讲述的内容是如何优化一个已经完成了基本功能的项目。在这个思路里，优化存在于方方面面，基本要涉及到以下几个原则：
1. 支持平稳退化
2. 降低JavaScript与HTML的耦合度
3. 减少先入为主的假设
4. 代码结构优化
<!--more-->
## 支持平稳退化
实际上就是，在JavaScript被禁用的环境下，使得项目同样能够正常工作。
**做法：为`a`标签填写真实存在的链接，而非不要使用空链接`#`和`javascript:`伪协议**

## 降低JavaScript和HTML的耦合度
在标签中给它添加点击事件是不好的实践。因此应该让HTML引用外部的脚本文件来完成这件事。

### 检查点
首先进行事件绑定应该想到如下几点：
1. 嗅探浏览器是否支持所要使用的DOM方法
2. 要调用DOM方法的元素是否存在
为了解决上面两个问题，可以用
{% codeblock lang:javascript %}
if (!document.getElementById) {
    return false;
}

if (!document.getElementsByTagName) {
    return false;
}

if (!document.getElementById("imagegallery")) {
    return false;
}
{% endcodeblock %}
来判断相关的DOM方法和元素是否存在。

### 改变行为
**注意：**为了在支持JavaScript的浏览器中，屏蔽链接的默认行为，在事件处理函数中我们应该添加`return false;`。例如：
{% codeblock lang:javascript %}
links[i].onclick = function() {
    showPic(this);
    return false;
}
{% endcodeblock %}

### `onload`事件共享
如果有一个方法，需要在文档加载结束之后立即运行，我们一般要做的是：
{% codeblock lang:javascript %}
window.onload = functionName();
{% endcodeblock %}
但是如果有一系列方法需要在文档加载结束之后依次执行呢？  
一个最基本的想法是把所有要执行的方法放在一个方法里，统一调用。但这种方法只适用于方法数目较少的情况。  
还有一种方法叫做`addLoadEvent`，它完成的事是：
1. 检查`onload`事件处理函数中是否有要执行的方法（是否已经被指定为一个函数）
2. 如果有，则将新函数添加到原来执行的函数后面
3. 如果没有，则将新函数指定为它的内容
{% codeblock lang:javascript %}
function addLoadEvent (func) {
    var oldload = window.onload;
    if(typeof oldload !== 'function') {
        window.onload = func;
    } else {
        window.onload = function() {
        oldload();
        func();
        }
    }
}
{% endcodeblock %}

## 减少先入为主的假设
这里主要指的是，在调用各种DOM方法时，应该首先对目标对象的存在性进行判断。如果不存在，也不至于在运行时报错，而应有其他方案来处理这个问题。比如`return false;`也是一种处理方式，它什么都不做但可以使程序终止。

## 代码结构优化
简单的`if`语句可以用三元表达式来代替。
**注意：`element.nodeName == "IMG"`其中`nodeName`这个属性总是返回大写字母。**
