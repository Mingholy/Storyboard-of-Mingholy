---
title: 编写可维护的JavaScript-第二部分-III
date: 2016-03-22 20:01:08
cover: '4.jpg'
category: "JavaScript"
tags:
    - JavaScript
---
>这一部分的主题是编程实践。主要的内容包括：
* 配置数据分离：将配置数据从代码中抽离，不必在每次修改代码的同时再去修改配置
* 抛出自定义错误：这是为了提高找到bug、纠正bug的效率

<!--more-->

## 分离配置数据
其实这一部分的核心思想还是解耦合。有些数据，尤其是可能直接面向用户的数据，是有很大几率被频繁修改的。如果这个数据在代码中被多次引用，那么要修改这个数据，就需要把所有的引用都改一遍，这是非常容易被遗漏的。  
首先，配置数据是应用中写死的值。它包括但不限于：
* URL
* 面向用户的字符串
* 重复的值
* 设置信息
* 以及任何可能发生变更的值

意思就是它们的出现之处，是在代码中固定的，然而它们的值却可能经常改变。  
首先看一个例子：
{% codeblock lang:javascript %}
function validate(value) {
    if (!value) {
        alert("Invalid value!");
        location.href = "/errors/invalid.html";
    }
}

function toggleSelected(element) {
    if (hasClass(element, "selected")) {
        removeClass(element, "selected");
    } else {
        addClass(element, "selected");
    }
}
{% endcodeblock %}
其中，``"Invalid value!"``是面向用户的字符串，URL是可能频繁修改的地址，修改类的方法中`selected`是CSS类名，这也可能遭到各种修改，而它居然出现了三处。当工程很大到时候，修改一个类名要付出多少代价可想而知。  
所以我们要对它进行重构，将可能被修改的配置数据抽离出来，做到低耦合，将数据与逻辑隔离。
{% codeblock lang:javascript %}
//create a config variable
var config = {
    MSG_INVALID_VALUE: "Invalid value!",
    URL_INVALID: "/errors/invalid.html",
    CSS_SELECTED: "selected"
};

//original function
function validate(value) {
    if(!value) {
        alert(config.MSG_INVALID_VALUE);
        location.href = config.URL_INVALID;
    }
}

function toggleSelected(element) {
    if (hasClass(element, config.CSS_SELECTED)) {
        removeClass(element, config.CSS_SELECTED);
    } else {
        addClass(element, config.CSS_SELECTED);
    }
}
{% endcodeblock %}
需要说明的是，`config`中各种数据的写法，比如包含信息的变量使用`MSG`作为前缀，URL地址和CSS各有相应的写法。上述重构就做到了不论数据内容如何修改，我们都不用去动逻辑代码，使用`config`对象的属性作为占位符来实现数据替换。  
更进一步，这些数据也不应该存在于一般的逻辑代码之间，它们应该有自己专属的位置，比如一个单独的文件中。最自然的想法是新建一个JavaScript文件来存放它们。但是有一个问题：这些数据必须符合JavaScript语法规范，如果一个地方不小心出现了错误，那么当把所有JavaScript文件合并在一起的时候，整个程序都会崩溃。而无论是在此之前将数据格式化以符合JavaScript标准，还是出错之后的debug，都非常麻烦。  
解决方法是，使用Java属性文件。  
类似于其他*主流*的编程语言，这里的数据形式为名值对，即：
{% codeblock lang:javascript %}
# message to user
MSG_INVALID_VALUE = Invalid value!

#URLs
URL_INVALID = /errors/invalid.html

#CSS Classes
CSS_SELECTED = selected
{% endcodeblock %}
有第三方工具，可以将这类Java属性文件转化为JavaScript能够接受的文件类型。包括
1. JSON
{% codeblock lang:javascript %}
{
    "MSG_INVALID_VALUE": "Invalid value!",
    "URL_INVALID": "/errors/invalid.html",
    "CSS_SELECTED": "selected"
}
{% endcodeblock %}
2. JSONP(JSON with padding)
{% codeblock lang:javascript %}
myfunc({"MSG_INVALID_VALUE": "Invalid value!", "URL_INVALID": "/errors/invalid.html", "CSS_SELECTED": "selected"});
{% endcodeblock %}
    这是将JSON结构用一个函数调用包装起来。
3. 纯JavaScript
将JSON赋给一个变量。使用的时候调用该变量。
{% codeblock lang:javascript %}
var config = {
    "MSG_INVALID_VALUE": "Invalid value!",
    "URL_INVALID": "/errors/invalid.html",
    "CSS_SELECTED": "selected"
}
{% endcodeblock %}

书中介绍的工具是作者自己编写的：[Props2Js](https://github.com/nzakas/props2js)
具体用法可参见项目文档。

## 抛出自定义错误
曾经跟着师兄照猫画虎写过几行Java。当时并没有学习过，一直也不懂那许许多多方法中动不动就是try throw catch的究竟是在做什么。甚至自己写的时候也只是格式上模仿了一下，根本没有搞懂这些代码究竟是在干什么。  
而现在看来，首先当时的一个逻辑就是错误且幼稚的：
我知道那些语句是跟出错有关的，大致意思是，如果没有出错，就继续向下执行；如果出错了，就要执行`throw`抛出一个异常信息。当时觉得错误不是在控制台给出的吗？而且在没执行代码之前，我知道能有什么错误？这样做有何意义？  
现在按照书中的原话可以更好地描述抛出错误信息的必要性：
> 一旦搞清楚了这一点（代码中哪里适合抛出错误），调试代码的时间将大大缩短，对代码的满意程度将急剧提升。

### 错误的本质
之前之所以会那么想，就是因为对错误本身的理解太过浅薄了。虽然发生错误本身就是个意外（相信没有多少人会故意去在代码中夹杂自己明知的错误），但是绝大大多数错误都是可以修补的。问题其实并不出在出错上，而是出在如何修补错误上。很多时候由于错误本身的意外性，它能报出的错误信息也是有限的。这样假如一个错误真的没有什么信息可以报给编写者，这要是让他来debug，岂不是要花费很多时间。一个有经验的程序员可能幸运地靠着见多识广解决它，但是初学者或者干脆这些代码就是别人写的而由我来调试，这些情况就十分痛苦了。
JavaScript的错误消息太过简洁，信息很少。而在这里我们自定义的错误信息，就是要弥补这种信息不足，使我们更加**清楚**程序是在哪里出现了错误。一个老师给我们讲过一个笑话：你们写出的代码，在两年之内只有你和上帝能看得懂；两年之后就只有上帝能看得懂了。为了避免这种情况就是我写这些东西的原因了。
这样做的目的是在出错的时候，能够有一些由你编写的，你能看得懂并且清楚原因的错误信息发送给你。这样这个错误就更容易被修复。

### 抛出错误
`throw`操作符可以将一个**任意对象**作为错误抛出。一般情况下是Error对象。Error对象的构造器接受一个字符串作为参数，也就是需要抛出的信息。
{% codeblock lang:javascript %}
throw new Error("Something goes wrong.")
{% endcodeblock %}
> 注意：不应该直接抛出字符串如：`throw "Something goes wrong."`因为部分浏览器不会接受这个字符串对象，而显示"Uncaught exception"

实际上抛出任何一个值，都将引发一个错误。虽然有些浏览器使用`String()`函数，至少能把信息显示出来，但是有些浏览器还是不会这样做。所以使用Error对象成了避免错误的必要手段。

### 好处
一个简单的函数：
{% codeblock lang:javascript %}
function getDivs(element) {
    return element.getElementByTagName("div");
}
{% endcodeblock %}
如果给该函数传递`null`参数，它将报错："object expected"。你并不知道究竟是哪个函数哪里出现了问题。
而如果这样做：
{% codeblock lang:javascript %}
function getDivs(element) {
    if (element && element.getElementByTagName) {
        return element.getElementByTagName("div");
    } else {
        throw new Error("getDivs(): Argument must be a DOM element.");
    }
}
{% endcodeblock %}
这就在出错的时候提供了两个信息：是哪个函数发生了错误，为什么发生了错误。所以作者建议在编写错误信息的时候一定要带上所在函数的名称。  

### 合适抛出错误
这里需要注意的是，我们只应该在最有可能引发错误的地方，设置错误检查。
如果一个函数，会由谁调用实现我们已经十分清楚，那么错误检查并不是必需的。如果一个函数我们事先并不能确定它被调用的所有情况，那么就需要一些错误检查了。抛出错误最佳的地方是工具函数中，也就是最容易被频繁调用的那些函数，包括一些JavaScript类库。
一些原则：
* 在修复了一个很难调试的错误之后，对它增加一个自定义错误，之后的调试就会因此而变得较为容易。
* 抛出错误的实际可以这样确定：在某处我希望某个情况不应该发生，如果发生了必出错无疑。这时就需要一个抛出错误。
* 为了其他人的读/写，根据他们的习惯，在特定的情况下抛出错误

> 抛出错误不是为了防止错误发生，而是在它们出现的时候更容易调试。

### `try-catch`语句
`throw`是创建并抛出一个自定义错误对象。
`try-catch`是将可能发生错误的代码放在`try`块内，如果它发生错误，立即停止执行，将错误对象传给`catch`。这个错误对象中含有错误类型、错误代码以及其他根部不同浏览器而不同的错误信息。`catch`块中的代码负责处理这个错误对象，例如显示错误信息，原因等。
二者结合我们将能够根据不同的状况抛出不同的自定义错误。
{% codeblock lang:javascript %}
function myFunction() {
    try {
        var x = document.getElementById("demo").value;
        if(x=="") throw new Error("Empty value");
        if(isNaN(x)) throw new Error("Non-number");
        if(x>10)     throw new Error("Too big");
        if(x<5)      throw new Error("Too small");
    } catch (err) {
        var y=document.getElementById("mess");
        y.innerHTML="Error：" + err + ".";
    }
}
{% endcodeblock %}
需要注意的是，绝对不要将`catch`块留空。即使只抛出错误信息，那也是在处理错误。不能什么都不做忽略错误。

### 错误类型
上边提到Error对象包含了错误类型的信息。JavaScript中的错误类型包括：
* Error
    基本类型...基本不会抛出的类型
* EvalError
    通过`eval()`执行代码时发生错误
* RangeError
    数字超出它的边界，例如`new Array(-20)`
* ReferenceError
    期望的对象不存在，例如试图调用`null`对象的函数
* SyntaxError
    `eval()`执行的代码中有语法错误
* TypeError
    变量不是期望的类型
* URIError
    encodeURI(),encodeURIComponent(),decodeURI(),decodeURIComponent()参数不合法

一个错误对象err，它可能是上述7中类型之中的一种。因此我们可以通过判断它的类型，来相应地采取措施处理错误。
{% codeblock lang:javascript %}
try {
    //code may cause trouble
} catch (err) {
    if (err instance of TypeError) {
        //code dealing with TypeError
    } else if (err instance of ReferenceError) {
        //code dealing with ReferenceError
    } else {
        //code dealing with other error type
    }
}
{% endcodeblock %}
这些是已经定义好的错误类型，有时候我们难以区分这样的一个错误究竟是我们设置好抛出的，还是浏览器抛出的。而自定义错误又丢失了一些浏览器给Error对象附加的一些重要信息，比如行号列号、执行栈和源码信息等。解决办法是创建自己的错误类型，并让它继承自Error。
{% codeblock lang:javascript %}
function MyError(message) {
    this.message = message;
}

MyError.prototype = new Error();
{% endcodeblock %}
后面的一个语句的意义是，将`MyError`这个构造函数的原型属性设置为Error的一个实例，这样由它所创建的实例对象就是一个错误对象，并且继承了Error对象的其他属性。

这一部分的内容是比较tricky的。灵活处理错误是一门艺术。我们需要敏锐的嗅觉和良好的习惯，以及实践积累下的经验，才能尽可能多地设置、捕获自己抛出的错误。这对形成优良而稳定的代码极其有益。
