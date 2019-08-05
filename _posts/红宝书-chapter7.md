---
title: "JavaScript函数与函数表达式"
date: 2017-03-23 11:14:00
cover: "8.jpg"
category: "JavaScript"
tags:
    - JavaScript
---
>关键知识点
  * 闭包
  * IIFE
  * 变量私有化

# 函数表达式
函数提升的概念前面的总结已经提到过。

<!--more-->
## 递归
这里需要注意的一点是，我们一直以来都是通过函数名称来调用自身。但是经常会出现我们把函数作为对象赋值给另一个变量。该变量存储的仅仅是指向该函数的指针。之后如果我们执行对函数名赋值（就是把函数名作为一个变量，比如赋值成`null`），那么所有指向这个函数的变量都不能执行函数了。

`arugments.callee`指向当前执行的函数；`arugments.caller`指向函数调用方。严格模式下它们不能访问，可以使用命名函数表达式：
{% codeblock lang:javascript %}
var factorial = (function f(num){
    if (num <= 1) {
        return 1;
    } else {
        return  num * f(num - 1);
    }
})
{% endcodeblock %}

## 闭包
进阶理解闭包和作用域链。
1. 全局作用域链是一直存在的，而拒不作用域链则是在函数被调用时创建的。
2. 作用域链本质上是一个指向变量对象的指针列表。
3. 对象的作用域链被保存在内部的`[[scope]]`属性中。全局作用域一直存在，它的`[[scope]]`存储的作用域链就包含了所有全局变量对象。
4. 在调用某个函数时，会为该函数创建一个执行上下文，先将全局对象的`[[scope]]`属性复制过来，然后再在其中添加该函数的活动对象。如果该函数还包含另一个函数B，那么在调用B时也会为它创建相应的作用域链。
5. 作用域链值引用但不包含变量对象。

以前看过的闭包介绍中说，“闭包能够将一些变量持续保存在内存中”。开始还不知道为什么，现在知道了内存回收机制和作用域链后可以尝试解释一下了。

首先JavaScript的内存回收包含两个机制，一是标记清除，二是引用计数。这两个算法都是基于“资格”的。未被标记或引用计数不为0，都代表该变量还有资格存在于内存中。

其次，如果闭包中引用了它外部函数的一些变量，那么
1. 在创建该闭包（一般就是个匿名函数）的作用域的时候，由于它引用的这些变量进入了该作用域，于是它们的标记被清除，暂时不会被回收，故可以持久化；
2. 在外部函数运行完毕后，外部函数的作用域链会被销毁，但是由于它的活动对象正在被闭包引用，此时它们的引用计数不为0，因此也不会被回收。
3. 匿名函数运行完毕被销毁之后，其他受它引用的变量才失去了驻留内存的资格，会被回收。

以前理解的作用域链要做出一些改变了。

之前的理解是，整个程序维护一个作用域链，调用一个函数就在作用域链前端添加一个节点，该函数中进行左右搜索的时候就在当前的作用域链前端开始，向后搜索。

实际情况是，每个执行上下文都有自己的作用域链，尽管它们可能有很多重复的内容，比如函数B在A内部，C在B的内部，C的
执行上下文创建的作用域链从后到前包括全局、A、B和C自己的活动对象；B的执行上下文包括全局、A和B自己的，等等。

需要注意的是：“闭包只能取得包含函数中任何变量的最后一个值。”
例子非常经典：
{% codeblock lang:javascript %}
function createFunction() {
    var result = new Array();

    for (var i = 0; i < 10; i++) {
        result[i] = function() {
            return i;
        };
    }
}
{% endcodeblock %}
在这里`result`是一个函数数组。但是需要注意的是，闭包的作用域链中，保存的`createFunction()`活动对象，是保存的引用。也就是说十个闭包引用的都是同一个`createFunction()`中的`i`。这个`i`从1变到10，每变一次，所有函数中的`i`都会同步变。

解决方式是“参数绑定”：
{% codeblock lang:javascript %}
...
        result[i] = function(num) {
            return function() {
                return num;
            };
        }(i);
...
{% endcodeblock %}
其中`function(num)`是匿名函数声明，函数体后面的(i)就是函数调用并传参数了。这是因为函数传参是**按值传递**，每个`i`会复制一份值给`num`并保存在当前执行上下文的作用域链的活动对象中。

## `this`对象
* 匿名函数的`this`对象一般会指向`window`
* `call()`和`apply()`会改变`this`的指向
* 使用闭包应该格外注意`this`的指向

## 内存泄漏
闭包的作用域链不应包含HTML元素。应该尽量把有用的信息复制出来，不让闭包直接引用HTML元素对象及其属性。否则该元素对象将无法被销毁，一直占用内存。

## 模仿块级作用域——IIFE
需要注意的有：
1. 重复声明一个变量，不会改变它的值（初始化会）
2. IIFE：immediately-invoked function expression:立即执行函数表达式，它会创建一个块级作用域，用来封装一些私有变量：
  {% codeblock lang:javascript %}
  //错误的写法
  function() {
      //块级作用域
  }();

  //正确的IIFE
  (function() {
      //块级作用域
  })();
  {% endcodeblock %}
3. 函数执行完毕立即销毁作用域链。减少内存消耗。

## 私有变量
新概念：**特权方法**
这个和Linux内核的那个特权级能类比么？暂且一看。

特权方法指的是在一个对象中，只有该方法能够访问到对象的某些成员变量，而其他任何方式都不可以。

在这个功能上，它有点像Crockford提出的稳妥构造函数模式。但是稳妥构造函数不使用`this`对象。该方法在构造函数中使用`this`添加访问成员变量的方法。而成员变量的声明则不用`this`。这样就不能直接修改成员变量，而只能通过对象中的闭包来访问了。
{% codeblock lang:javascript %}
function Person() {

    this.getName = function() {
        return name;
    };

    this.setName = function (value) {
        name = value;
    };
}
{% endcodeblock %}
缺点显而易见：构造函数中的两个方法会在每个实例中重新创建。

### 静态私有变量
这种方式可以解决无限复制分发的问题。（不知道这么描述对不对）

这种模式使用全局块级作用域包含闭包来实现方法公有化，成员变量私有化。
{% codeblock lang:javascript %}
(function() {
    var name = ";

    Person = function(value) {
        name = value;
    }

    Person.prototype.getName = function() {
        return name;
    };

    Person.prototype.setName = function(value) {
        name = value;
    };
})();
{% endcodeblock %}
需要说明的是：所有实例共享一个静态变量`name`和两个原型方法，`setName`和新建实例传不同的值，都会修改所有实例的`name`属性。

### 模块模式
为单例创建私有变量和特权方法的模式。
{% codeblock lang:javascript %}
var singleton = function() {

    //私有变量和函数
    var privateVariable = 10;

    function privateFunction() {
        return false;
    }

    //特权方法，公有方法和属性
    return {
        publicProperty : true;

        publicMethod : function() {
            privateVariable++;
            return privateFunction();
        }
    }
})
{% endcodeblock %}
这种模式适合创建一个全局对象，该对象中需要初始化一些数据，还要提供访问这些数据的公共方法。
注意：
* 创建的每个单例都是`Object`的实例，没有自定义类型
* 通过匿名函数建立，赋值给一个对象字面量

### 增强模块模式
如果需要使单例称为某个自定义类型的实例，需要在上面的单例函数中，`new`一个指定自定义类型的实例，再把各种属性和方法挂载到这个实例上。这样返回的对象就既有私有属性、公共接口，同时又是某个自定义类型的实例：
{% codeblock lang:javascript %}
var application = function() {

    //私有变量和函数
    var components = new Array();

    //初始化
    components.push(new BaseComponent());

    //创建application的一个局部副本
    var app = new BaseComponent();

    //公共接口
    app.getComponentCount = function() {
        return components.length;
    };

    app.registerComponent = function(component) {
        if (typeof component == "object") {
            components.push(component);
        }
    };

    return app;
}();
{% endcodeblock %}
