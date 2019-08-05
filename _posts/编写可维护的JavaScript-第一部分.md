---
title: '编写可维护的JavaScript:第一部分'
cover: "2.jpg"
date: 2016-03-07 19:03:54
category: JavaScript
tags:
    - JavaScript
---


## 基本格式化
###  缩进
4空格为一个缩进单位。
###  语句结尾
自动分号插入机制 ASI 会在大多数应该加分号的地方加分号但是这种自动机制可能会引起某些意外的错误。因此还是**手动加分号**，也不要省略分号。
###  行的长度
一般是80个字符的限制。  
###  换行
需要拆行的时候，在运算符后拆行，并且在拆行之后设置两个缩进。
如果是if语句这种，比如在判断条件语句拆行，它之后的执行主体只设置一个缩进。  
> 例外:变量赋值时，第二行的位置应该和赋值运算符的位置保持对齐。  

<!--more-->
###  空行
* 方法之间
* 方法中的局部变量和第一条语句之间
* 多行或者单行注释之前
* 方法逻辑片段之间

###  命名方法
一般是小驼峰(Camel Case)。首字母大写的叫做大驼峰(Pascal Case)。  
* 变量与函数  
can:函数返回一个布尔值  
has:函数返回一个布尔值  
is:函数返回一个布尔值  
get:函数返回一个**非**布尔值  
set:函数用来保存一个值    
* 常量  
大写字母命名，下划线分隔单词。  
* 构造函数  
使用大驼峰命名法，且其之前需要有`new`。  

### 直接量
*  字符串  
对字符串的描述，使用双引号和单引号的作用是一样的。但是注意双引号所引用的字符串中的双引号需要转义。**最好保持一致的风格**。  
多行字符串使用`＋`。  
{% codeblock lang:javascript %}
var longString = "Here's a string with " + "two parts."
{% endcodeblock %}

*  数字  
整数部分为0的浮点数不推荐省略整数部分的做法。
*  `null`  
  什么时候应该用`null`？
    1. 初始化一个变量，它可能成为一个对象;
    2. 用来和一个 已经初始化的变量比较，该变量可以是也可以不是一个对象:
    3. 函数参数是对象，用作参数传入;
    4. 函数参数是对象，用作返回值传出。

  什么时候不应该使用`null`？
    * 不要用`null`来检测是否传入了某个参数。    

    它只是一个对象的占位符。  
*  `undefined`  
    虽然
{% codeblock lang:javascript %}
null == undefined
{% endcodeblock %}
    的结果为真，但实际他们是不同的对象。  
    `undefined`表示一个没有被初始化的对象的初始值。当一个对象初始化为`null`时，它的类型为`object`，即`typeof`为`object`。

*  对象直接量  
    声明一个对象并对齐多个数属性初始值时，不应将属性使用多个语句单独赋值。
{% codeblock lang:javascript %}
var book = {
    title: "Maintainable JavaScript",
    author: "Nicholas C.Zakas"
};
{% endcodeblock %}

*  数组直接量
类似于对象直接量，数组直接量也可以统一声明多个值。
{% codeblock lang:javascript %}
var colors = ["red","green","blue"];
{% endcodeblock %}

## 注释
### 单行注释
用法通常是这样的:  
1. 独占一行，注释对象是下一行代码。该行注释之前应该有一个空行，且缩进层级和下一行代码一致;
2. 代码尾部注释，该注释与代码结束之间留有至少一个缩进的距离;
3. 注释大段代码。  

### 多行注释
以/\*注释内容\*/来表示注释起始和结尾。多行注释中每一行左侧都行有一个\*。在注释之前也需要有一个空行并且应该保持缩进的一致性。
> 代码尾部的注释不应用多行注释。

### 使用注释
原则:在有需要的情况下使用注释，包括但不限于标明某段代码的特殊含义或者非常重要的片段等:  
1. 难以理解的代码
2. 可能被误认为错误的代码
3. 浏览器特性相关

## 语句和表达式
> 任何时候都不应该省略块语句首尾的花括号。  

正确的写法应该是
{% codeblock lang:javascript %}
if (condition) {
    doSomething();
}
{% endcodeblock %}
### 花括号的对齐方式
{% codeblock lang:javascript %}
if (condition) {
    doSomething();
} else {
    doSomethingElse();
}
{% endcodeblock %}
### 块语句间隔
{% codeblock lang:javascript %}
if (condition) {
    doSomething();
}
{% endcodeblock %}
### Switch
#### 缩进
{% codeblock lang:javascript %}
switch(condition) {
    case "first":
        //code
        break;  

    case "second":
        //code
        break;  

    default:
        //code
}
{% endcodeblock %}
或者将`case`与`switch`保持相同缩进。
#### 连续执行case
{% codeblock lang:javascript %}
switch(condition) {

    //excute in sequence
    case "first":
    case "second":
        //code
        break;

    case "third":
        //code

        /* fall through */
    default:
        //code
}
{% endcodeblock %}
### for与for-in
原则上类似于其他块级代码语句。

## 变量、函数和运算符
### 变量声明
需要注意的是，代码中所有已声明和未声明的变量，在解释器执行的时候都会补全成`var`语句，放到包含这段逻辑的函数的顶部执行。  
最终实际运行的可能是这个样子的:
{% codeblock lang:javascript %}
function doSomething() {

    var result;
    var value;

    result = 10 + value;
    value = 10;

    return result;
}
{% endcodeblock %}
所以为了避免引起不必要的误会，坦白地说就是以提高代码可读性为最终目的，应该将所有变量的声明提前到所在逻辑段的顶部，使用单`var`语句，每个变量的初始化独占一行，未初始化变量后置。  
{% codeblock lang:javascript %}
function doSomethingWithItems(items) {

    var value = 10,
        result = value + 10,
        i,
        len;

    for (i = 0, len = items.length; i < len; i++) {
        doSomething(items[i]);
    }
}
{% endcodeblock %}
### 函数声明
与变量类似，JS引擎也会将函数声明提前。所以应该养成一个习惯：先声明后使用。  
函数内部的局部函数应当紧接着变量声明之后声明。
{% codeblock lang:javascript %}
fucntion doSomethingWithItems(items) {

    var value = 10,
        result = value + 10,
        i,
        len;
    function doSomething(item) {
        //code
    }

    for (i = 0, len = items.length; i < len; i++) {
        doSomething(items[i]);
    }
}
{% endcodeblock %}
> 注意：函数声明不应出现在语句块中。比如在`if`代码块中。
{% codeblock lang:javascript %}
if (condition) {
    function doSomething() {
        alert("Hi!");
} else {
    function doSomething() {
        alert("Yo!");
    }
}
{% endcodeblock %}

### 函数调用间隔
函数调用与块语句声明不同，在函数名和左括号之间没有空格。
{% codeblock lang:javascript %}
doSomething(item);
{% endcodeblock %}
### 立即调用的函数
需要注意的是，匿名函数本身可以赋值给一个变量。在函数体后添加一对括号可以使之立即执行并返回一个值，将此值赋给变量。  
但是并不建议直接在函数体后面添加括号来获取一个返回值。而是应该在函数体前后都加上括号，以区别函数赋值与返回值赋值。
{% codeblock lang:javascript %}
// function assignment
var doSomething = function() {
    //code
};
// return value assignment
vae doSomething = (function(){

    //code

    return {
        message: "Hi!"
    }
}());
{% endcodeblock %}
### 严格模式
`"use strict"`
指令可以使脚本以严格模式运行。  
注意，不要轻易使用全局`"use strict"`。因为在一个文件中使用全局`"use strict"`，在多个文件相连时，作用于会扩展到所有文件。这会引起意料之外的错误。  
因此最好使用局部严格模式。在单个函数中使用，或者将需要使用严格模式的多个函数包裹在立即执行的函数之内。
{% codeblock lang:javascript %}
// single function strict mode
function doSomething() {
    "use strict";
    //code
}
// multi function strict mode
(function (){
    "use strict";
    function doSomething() {
        //code
    }
    function doSomethingElse() {
        //code
    }
})();
{% endcodeblock %}
### 相等
首先在判断相等上存在的问题是，JavaScript存在一个强制类型转换的机制。当使用`==`和`!=`的时候，引擎总是要判断它们是否是同一类型，并且如果不是同一类型，将会改变其中至少一个对象的类型，再进行比较。比如
{% codeblock lang:javascript %}
5 == "5"
{% endcodeblock %}
的值为`true`。这种情况不局限于数值与字符串比较，还存在于不同进制的数值，数值与布尔值，对象之间的比较。两个特殊的栗子:
{% codeblock lang:javascript %}
console.log(object  == 25); // true
console.log(null == undefined); //true
{% endcodeblock %}
而这显然不是我们想要的。因此最好使用`===`和`!==`。它不涉及到强制类型转换，也就是说如果比较双方的类型不同，那么它们就是不同的。
#### eval()
`eval()`，`setTimeout()`，`setInterval()`这一类函数的特点是，它们的参数是一个字符串，它们会将传入的字符串当做JavaScript代码执行。  
一般情况下不应使用`eval()`，除非在涉及回调解析JSON和将Ajax返回值转换为JavaScript值的情形下。  
而`setTimeout()`和`setInterval()`则是不允许使用字符串作为参数，而是应该使用立即执行的函数对象作为它们的参数。
#### 原始包装类型
对一个原始类型使用某一特定类的方法，比如对一个字符串原始类型的变量，使用`String`类的方法，实际上也可以用，但是它的过程是，在调用类方法的时候，先创建`String`类型的新的临时实例，执行完类方法之后再销毁该临时实例。  
不建议使用类似
{% codeblock lang:javascript %}
var name = new String("Nicholas");
var author = Boolean(true);
{% endcodeblock %}
这样的声明方法。


第一部分 编程风格到此结束，第二部分是编程实践。
