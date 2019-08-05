---
title: "JavaScript DOM 编程艺术 笔记-第二章"
date: 2017-01-24 20:00:00
cover: '3.jpg'
category: "JavaScript"
tags:
    - JavaScript
---
>虽然大多数都是最简单的内容，但是为了养成良好的编码习惯，需要注意的还是要记下来。总没有坏处。

##
{% codeblock lang:javascript %}
var foo = "first statement";    //最好在每一行后都加上分号，单行注释使用双斜杠
/* 多行注释使用
   这种方式比较好 */
var mood = "happy", age = 23;  //变量与其他语法元素的名字都是区分大小写的。
var myMood = "happy";  //驼峰命名
{% endcodeblock %}
* 字符串最好用双引号包裹。不管是单引号还是双引号都可以，但必须要全文同一。
* 由浏览器提供的预定义对象被称为是**宿主对象**，如Form、Image和Element等，以及最著名的document。
* 下面这个例子比较特殊（来自译注）：
{% codeblock lang:js %}
if (a = false) {
    alert('hello, world');
}
{% endcodeblock %}
这里作者是想说明，不应以一个赋值语句作为`if`的判断条件，因为赋值语句总是会成功的。但事实上这里的`a = false`虽然是成功的，但是它并没有返回`true`而是返回了`false`。这里的返回值是在赋值成功后变量的值。  
另外，在之前所写的《编写可维护的JavaScript》笔记中也提到，强判断两个变量是否相等，应使用`===`作为类型与值双重判断的操作符。
