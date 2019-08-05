---
title: 编写可维护的JavaScript-第二部分-IV
date: 2016-03-24 20:01
cover: '4.jpg'
category: "JavaScript"
tags:
    - JavaScript
---

>这一部分的主题是编程实践。主要的内容包括：
* 对象持久化：不是你的对象不要动，这里涉及到多个有关方法的原则，以及关于继承的处理
* 浏览器嗅探：实际上这是不推荐的行为

<!--more-->
## 不是你的对象不要动
前面的三个部分里或多或少已经提到所谓的全局对象污染、命名空间的重要性，这些其实都在强调一点：
> JavaScript的独一无二之处在于任何东西都**不是**神圣不可侵犯的。

因为解析器不会在意对象究竟来自哪里。

### 原则
如果一个对象并非由你所有，不要修改它。包括：
* 原生对象
* DOM对象(document)
* 浏览器对象模型(BOM)对象(window)
* 类库的对象

#### 不覆盖方法
在JavaScript中即使是`document.getElementById()`这样的方法也可以轻易地被覆盖。我的意思是，不要以原生的方法名字来定义你自己的方法。即使形如下面的做法，也不应该出现。
{% codeblock lang:javascript %}
document._originalGetElementById = document.getElementById;
document.getElementById = function(id) {
    if ( id == "window") {
        return window;
    } else {
        return document._originalGetElementById(id);
    }
};
{% endcodeblock %}
这样的做法虽然将原生方法的指针保存了下来，但是`document.getElementById`方法仍然被覆盖了。这样的后果是，在使用这个方法时，会出现有时能得到正确结果，而有时又得不到正确结果。因此也不是一种好的实践。
这样的做法也被称作“函数劫持”。

#### 不新增方法
我们之前有提到过，其实所谓新增的最好途径是扩展原有功能。但是需要注意的是，如果一个对象是原生的或者是别人已经编写好的，我们不应该在这个对象上新增方法。因为这有很大的几率会导致命名冲突。而这个对象现在没有的方法并不代表它以后也不会有，这样冲突就更多了。
{% codeblock lang:javascript %}
document.sayImAwesome = function() {
    alert("You're awesome.");
};

Array.prototype.reverseSort = function() {
    return this.sort().reverse();
};

YUI.doSomething = function() {
    //code
}
{% endcodeblock %}
上述三个例子，都是不好的做法。第一个函数在DOM对象上增加了方法；第二个函数在原生对象Array上增加了方法。第三个函数在库对象上增加了方法。
#### 不删除方法
删除一个方法同样简单，只需要给这个方法复制为`null`就行了。即使确认需要弃用一个方法，也不应该直接删除它，而是在它上面做一个标记表明该方法已经被弃用。
### 更好的途径
上述各种问题和糟糕实践的解决方法，其实就是**继承**。
> 在JavaScript之外，最受欢迎的对象扩充的形式是继承。如果一种类型的对象已经做到了你想要的大多数工作，那么继承自它，然后再新增一些功能即可。在JavaScript中有两种基本的形式：基于对象的继承和基于类型的继承。

在JavaScript中，继承有一些限制，首先暂时不能从DOM或者BOM对象继承。其次由于数组索引和`length`属性之间复杂的关系，继承自Array也不能正常工作。
#### 基于对象的继承
基于对象的继承，也经常叫做**原型继承**。一个对象继承另外一个对象无需调用构造函数，`Object.create()`是最简单的方式。
{% codeblock lang:javascript %}
var person = {
    name: "Name1";
    sayName: function() {
        alert(this.name);
    }
};

var myPerson = Object.create(person);
myPerson.sayName();
{% endcodeblock %}
`myPerson`对象继承了`person`的两个属性，因此在它调用`sayName()`时也会返回Name1。这时如果重新定义这个方法，就不会返回Name1了。
`Object.create()`方法可以指定第二个参数，该参数对象中的属性和方法将添加到新的对象中。是为扩展：
{% codeblock lang:javascript %}
var myPerson = Object.create(person, {
    name: {
        value: "Name2"
    }
});

myPerson.sayName(); //Name2
person.sayName(); //Name1
{% endcodeblock %}
这种方式下创建的新对象，是完全由我们自己所有的，我们可以任意修改其中的内容，而不必担心造成什么重大的错误。
#### 基于类型的继承
这种继承模式依赖于原型。我还不清楚为什么不调用构造函数的基于对象的继承反而称为“原型继承”。基于类型的继承要通过构造函数实现，而非对象。
{% codeblock lang:javascript %}
function MyError(message) {
    this.message = message;
}

MyError.prototype = new Error();
{% endcodeblock %}
这里的`Error`称为超类。将`MyError.prototype`赋为一个Error实例，就代表每个MyError实例对象都将继承自Error，也是一个Error实例对象。
在开发者定义了构造函数的情况下，基于类型的继承是最合适的。基于类型的继承一般需要两步：
1. 原型继承
2. 构造器继承

构造器继承是调用超类的构造函数时，传入新建的对象作为其`this`的值。
{% codeblock lang:javascript %}
function Person(name) {
    this.name;
}

function Author(name) {
    Person.call(this, name);
}

Author.prototype = new Person();
{% endcodeblock %}
其中第二个函数就是构造器继承。
解释一下：`Author`类继承自`Person`，属性`name`是`Person`类管理的，构造器继承函数中`Person.call(this, name)`是明确由`Person`构造器持续定义该属性。但这个函数中的`this`是一个`Author`对象，因此`name`最终是定义在`Author`对象上的，继承自超类`Person`并且由超类来管理。这样做的好处是，我们可以定义一个公共的超类，若干个不同的类型具有相同的一些属性时，可以继承自该超类，而它们各自不同的属性，则可以单独地进行各自定义，这些属性与超类中的公共属性应该是完全不同的。

## 浏览器嗅探
虽然书中大体上观点是，不推荐进行浏览器嗅探，但是毕竟前端是要做浏览器兼容的，总要知道用户用的什么浏览器，才能进行相应的兼容性完善。
### User-Agent检测
不同的浏览器有不同的`user-agent`字符串，以作为区分浏览器的标识。由于一些说来话长的原因，`user-agent`字符串并不是总会准确地反映浏览器是什么，浏览器总要保证自己的兼容性，于是它们会相互复制对方的字符串加到自己的字符串里来，这样造成的混乱乃是冰冻三尺非一日之寒。然而为了保证JavaScript能够正确地运行，用户代理检测也是没有办法的办法。由于浏览器在更新换代时，经常更改/添加`user-agent`字符串，因此在进行用户代理检测的时候，检测旧版本就行了，因为它们的字符串已经不会再更改了。否则每更新一代浏览器，可能就需要更新一下用户代理检测的代码，不胜其烦。需要知道的是，所有的浏览器的`user-agent`字符串都可以修改。不过一般能够修改且已经修改了的用户，应该都知道自己在做什么。
### 特性检测
基于上述原因，另一种检测浏览器的方法是特性检测。这是根据浏览器的行为鉴别浏览器的方法。
通过`user-agent`可以得到浏览器的名称版本号，而特性检测只能确定某个方法或者对象是否存在。
{% codeblock lang:javascript %}
function setAnimation(callback) {
    if (window.requestAnimationFrame) {
        return requestAnimationFrame(callback);
    } else if (window.mozRequestAnimationFrame) {
        return mozRequestAnimationFrame(callback);
    } else if (window.webkitRequestAnimationFrame) {
        return webkitRequestAnimationFrame(callback);
    } else if (window.oRequestAnimationFrame) {
        return oRequestAnimationFrame(callback);
    } else if (window.msRequestAnimationFrame) {
        return msRequestAnimationFrame(callback);
    } else {
        return setTimeout(callback, 0);
    }
}
{% endcodeblock %}
上述代码首先探测标准的`requestAnimationFrame`方法是否存在，如果不存在再探测不同浏览器的方法。
### 避免特性推断
特性推断是指：尝试使用多个特性检测，但是只验证了其中之一。根据一个特性的存在来推断另一个特性是否存在。一个例子：
{% codeblock lang:javascript %}
function geById (id) {
    var element = null;

    if(document.getElementByTagName) {  //DOM
        element = document.getElementById(id);
    } else if (window.ActiveXObject) {  //IE
        element = document.all[id];
    } else {
        element = document.layers[id];
    }

    return element;
}
{% endcodeblock %}
这是一段很古老的代码了，最后一个`else`中是判断浏览器为Netscape4之前的浏览器。这个浏览器特性推断犯了若干逻辑上的错误：
* 如果`document.getElementByTagName()`存在，则`document.getElementById()`也存在。
    这明显是不合逻辑的，不能从一个DOM方法存在推断所有的方法都存在。
* 如果`window.ActiveXObject()`方法存在，则`document.all`也存在。
    也就是断定了`window.ActiveXObject`仅仅存在于IE中，且`document.all`也仅存在于IE。而Opera的部分版本也支持`document.all`。
* 如果上述检测均未通过，直接断定是Netscape Navigator 4之前的版本。
    这个判断更加武断。因为上面的检测都没有达到不重不漏，这里更不可能准确断定。

这是一个`伪鸭式辩型`。如果它看起来像一只鸭子，它必定像鸭子一样嘎嘎叫。而之前检测类型中所用的鸭式辩型是指，如果一个对象/类型存在某个专属于这个类型的方法，它就一定是这个类型。
### 避免浏览器推断
同样浏览器推断也是应该极力避免的。它同特性检测一样都存在很大的逻辑漏洞。这里就不再赘述
### 如何取舍
特性推断和浏览器推断应该极力避免。纯粹的特性检测能够准确地得到想要的结果。注意：只应检测特性是否可用，不要试图推断特性之间的关系，否则检测结果将不可靠。
使用用户代理检测唯一安全的场景是，针对老旧的浏览器。
因此作者的建议是尽可能地使用特性检测。如果对于老旧浏览器来说特性检测并不可用，可以使用用户代理检测。

第二部分结束。
