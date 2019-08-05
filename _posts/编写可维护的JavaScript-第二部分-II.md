---
title: 编写可维护的JavaScript-第二部分-II
date: 2016-03-15 17:35:47
cover: '4.jpg'
category: "JavaScript"
tags:
    - JavaScript
---
>这一部分的主题是编程实践。主要的内容包括：
* 事件处理：同样是传递参数与应用逻辑的解耦
* 避免“空比较”：即判定某一变量/对象是否是空，这里要分清各个对象的类型，不能混用

<!--more-->
## 事件处理
事件触发时，事件对象，即event对象会作为回调参数传入到事件处理程序中。event对象包含和事件相关的所有信息。而一般的时间处理逻辑，都只会用到event包含的一小部分信息。这时将event对象整个作为参数传进去，就不是一个好的做法。
以下面的程序为例：
{% codeblock lang:javascript %}
function handleClick(event) {
    var popup = document.getElementById("popup");
    popup.style.left = event.clientX + "px";
    popup.style.top = event.clientY + "px";
    popup.className = "reveal";
}

addListener(element, "click", handleClick);
{% endcodeblock %}
这段程序只用到了event对象的两个属性：`clientX`和`clientY`。这种做法的不好之处在于以下两个方面
### 没有隔离应用逻辑
首先需要明确什么是应用逻辑。
> 应用逻辑是和应用相关的功能性代码，而非和用户行为相关的。

也就是说，这里的应用逻辑应该是：在指定的位置显示一个弹出框。但是至于具体是用户点击时，还是键盘按键时，这与应用逻辑本身的实现*不应该*有关系。  
上述代码的第一个不好之处在于，`handleClick(event)`是一个事件处理函数，但是它却还包含了应用逻辑。这样假如我还需要在用户按键盘的时候同样显示一个弹出框，那么我还得编写功能相同内容类似的代码，只不过将事件绑定在某个按键上。于是多个事件处理程序执行同样的逻辑的时候，同样一个应用逻辑代码就被无形中复制了n多份。  
在这个大背景下，另一个不好之处就是难以测试。测试并不是模拟用户行为（即点击绑定元素），而是直接触发功能代码。如果像上面那样将事件处理和应用逻辑紧紧地耦合起来，那么测试这个应用逻辑的唯一办法就是模拟事件触发。而不幸的是，即使某些测试框架可以做到模拟事件触发，直接调用功能函数仍然是最好的方法。  
所以最佳实践就是，无论何时都要将应用逻辑和事件处理代码拆分开来。事件处理有事件处理的函数，应用逻辑有应用逻辑的函数。  
上述代码重构：
1. 将处理弹出框逻辑的代码放入单独的一个函数，该函数可能挂在与为该应用定义的一个全局对象上。
2. 事件处理函数应当总是在一个相同的全局对象中。

{% codeblock lang:javascript %}
var MyApplication = {
    handleClick: function(event) {
        this.showPopup(event);
    },

    showPopup: function(event) {
        var popup = document.getElementById("popup");
        popup.style.left = event.clientX + "px";
        popup.style.top = event.clientY + "px";
        popup.className = "reveal";
    }
};

addListener(element, "click", function(event) {
    MyApplication.handleClick(event);
});
{% endcodeblock %}
上面的重构代码中，应用逻辑在`MyApplication.showPopup()`中，事件处理在`MyApplication.handleClick()`中，这样对同一逻辑的调用（此处即`MyApplication.showPopup()`）可以在多处发生，也不依赖某个特定事件的触发。
### 过度分发事件对象
应用逻辑与事件处理解耦之后，另一个问题就出现了。这两者所需要的参数都是event对象。于是每次点击和调用功能代码的时候，event对象就被复制分发。而event对象包含了许许多多属性信息，上述功能只使用了其中的两个属性。书中的观点是：
> 应用逻辑不应当依赖于event对象来正确完成功能
* 方法接口并没有明确哪些数据是必要的。好的API一定是对于期望和依赖都是透明的。也就是说应该告诉我们传入的event对象，哪些属性是有用的。
* 又带来了测试难的问题。如果想测试这个方法，就必须重新创建应该event对象并将它作为参数传入。这样作为测试我们还得知道event中那些信息是有用的，才能构造正确的测试代码。

当然，大多数时候我们难以清楚地知道应用逻辑究竟是干了些什么。所以虽然明确接口需求是最好的，但是并不一定总能做到。因此这里的主要问题还是自行构造event对象造成的不容易测试。  
在知道应用逻辑方法只需要两个参数的时候，我们可以重构一下：
{% codeblock lang:javascript %}
var MyApplication = {

    handleClick: function(event) {
        this.showPopup(event.clientX, event.clientY);
    },

    showPopup: function(event) {
        var popup = document.getElementById("popup");
        popup.style.left = x + "px";
        popup.style.top = y + "px";
        popup.className = "reveal";
    }
};

addListener(element, "click", function(event) {
    MyApplication.handleClick(event);
});
{% endcodeblock %}
原则是，**处理事件的时候，最好让事件处理程序称为接触到event对象的唯一函数。**事件处理函数的其他职责还包括：阻止默认事件、阻止事件冒泡等。比如：
{% codeblock lang:javascript %}
var MyApplication = {

    handleClick: function(event) {

        //pre-proccessing
        event.preventDefault();
        event.stopPropagation();

        this.showPopup(event.clientX, event.clientY);
    },

    showPopup: function(event) {
        var popup = document.getElementById("popup");
        popup.style.left = x + "px";
        popup.style.top = y + "px";
        popup.className = "reveal";
    }
};

addListener(element, "click", function(event) {
    MyApplication.handleClick(event);
});
{% endcodeblock %}
这里的预处理就添加了阻止默认事件和事件冒泡的操作。
## 避免“”空比较”
在实际编写JavaScript的时候，会经常遇到判断一个对象，不管它是什么类型的，是否为空，是否已经被赋一个合法的值，其中是否存在一个我需要的属性值，等等情况，都需要使用逻辑运算符得到一个准确的结论。然而它并不一定真的准确，这还取决于运算符后面那个东西，是否用对了。
### 检测原始值
JavaScript有五种**原始类型**：
1. 字符串
2. 数字
3. 布尔值
4. `null`
5. `undefined`

运算符`typeof`会返回表示值的类型的字符串。
* 字符串："string"
* 数字："number"
* 布尔值："boolean"
* undefined："undefined"

实例：
{% codeblock lang:javascript %}
//check string
if (typeof name === "string") {
    //code
}

//check number
if (typeof count === "number") {
    //code
}

//check boolean and true
if (typeof found === "boolean" && found) {
    //code
}

//check undefined
if (typeof MyApp === "undefined") {
    //code
}
{% endcodeblock %}
其中，对于未声明的变量，它将返回`undefined`。  
至于原始值`null`,**它一般应用于检测语句。**因为一个对象如果不为`null`,但是它可能是其他任意一种类型，这样类似
{% codeblock lang:javascript %}
if (items !== null) {
    //code
}
{% endcodeblock %}
这样的判断语句仍然会通过，即使接下来的代码得到的并不是它所需要的对象。
> 注意，`typeof null`返回的结果是"object"。要检测`null`，需要使用恒等运算符`===`和非恒等运算符`!==`。

### 检测引用值
和上述的原始类型（原始值）区别的是“引用类型（引用值）”。在JavaScript中除了原始值之外，都是引用。包括：
1. Object
2. Array
3. Date
4. Error

`typeof`运算符检测这里的所有类型，包括之前的`null`，返回的都会是`object`，所以它不能用来检测这些类型。  
检测引用值类型的方法是使用`instanceof`运算符。基本语法：
{% codeblock lang:javascript %}
value instanceof constructor
{% endcodeblock %}
实例：
{% codeblock lang:javascript %}
//check Date
if (value instanceof Date) {
    console.log(value.getFullYear());
}

//check RegExp
if (value instanceof RegExp) {
    if (value.test(anotherValue)) {
        console.log("Matches");
    }
}

//check Error
if (value instanceof Error) {
    throw value;
}
{% endcodeblock %}
它的检测模式是，检测这个对象的构造器和原型链。因此可能出现的情况是，默认情况下每一个对象都继承自`Object`，因此`value instanceof Object`总会返回`true`。所以如果有更具体的引用类型信息，可以使用`instanceof`来检测某一对象是否是该引用类型。但是用它来检测`Object`就不太适合了。   
它也可以检测自定义类型，比如有构造函数：
{% codeblock lang:javascript %}
function Person(name) {
    this.name = name;
}

var me = new Person("Tom");

console.log(me instanceof Object); //true
console.log(me instanceof Person); //true
{% endcodeblock %}
由于变量`me`是自定义类型`Person`的实例，而所有的对象都被认为是`Object`的实例，因此下面两条判断语句都返回`true`。于是如果要检测自定义类型，`instanceof`是最佳且唯一的选择。  
需要注意的是，对于不同的浏览器帧，它们各自定义的构造函数可能具有相同名称，相同的定义，但是本质上是不同的。比如都定义了同样的构造函数`Person`，但是帧A中的实例对象，并不是帧B中`Person`的实例。因此使用`instanceof`进行交叉判断的时候，会返回`false`。
#### 检测函数
函数是一种对象。不过实际上它也是一种引用类型，它的构造函数是`Function`，声明的`function()`就是它的实例。但跟之前提到的一样，使用`instanceof`判断一个对象是否是`Function`的实例，也不能跨帧使用，因为每个帧都有自己的`Function`构造函数。不过这时可以使用`typeof`，**它可以跨帧使用，并且对函数对象返回字符串`function`。**可以使用如下方法判断：
{% codeblock lang:javascript %}
function myFunc() {}

//check Function
console.log(typeof myFunc === "function"); //true
{% endcodeblock %}
#### 检测数组
另一个涉及跨帧检测问题的对象是数组。`instanceof Array`不能返回正确的结果，跟上述原因一样，不同的帧有各自的`Array`构造函数。但是可以利用`Array`类型本身的特性来检测它的实例。这个方法称为“鸭式辩型”，即检测对象的`sort()`方法是否存在。
{% codeblock lang:javascript %}
function isArray(value) {
    return typeof value.sort === "function";
}
{% endcodeblock %}
这个方法依赖一个事实：数组是唯一包含`sort()`的对象。如果你创造了一个对象，也包含`sort()`那么上述方法检测这个对象也会返回`true`。  
最终的办法是比较通用而准确的。Kangax发现调用某个值的内置`toString()`方法会返回与其类型相关的标准字符串结果，比如对数组返回"[object Array]"，对JSON返回"[object JSON]"。不过这种方法仅限于内置对象，自定义对象还是不可用。
{% codeblock lang:javascript %}
function isArray(value) {
    return Object.prototype.toString.call(value) === "[object Array]";
}
{% endcodeblock %}
ES5中引入了一个新的方法：`Array.isArray()`，可以跨域使用。
### 检测属性
检测一个属性在对象中是否存在时，可以使用`null`和`undefined`。注意下列三个判断都存在局限性：
{% codeblock lang:javascript %}
if (object[propertyName]) {
    //code
}

if (object[propertyName] != null) {
    //code
}

if (object[propertyName] != undefined) {
    //code
}
{% endcodeblock %}
上述判断做到的不是判断给定名称所指值是否存在，而仅仅是检查该值内容。因为当属性值为假值时，结果都会出错，比如0，""（空字符串），false，null和undefined。但有些值是合法的，比如数字0。但在第一个判断语句中就会出错。  
判断**存在性**的最好方法是使用`in`运算符。它仅仅会简单滴判断属性是否存在，而不关心属性的内容。`in`返回"true"的情况包括该属性在实例对象中存在，或者继承自对象的原型。如果只想检查实例对象的某个属性是否存在，则需要使用`hasOwnProperty()`方法，这个方法是所有继承自`Object`的JavaScript对象都具有的。
{% codeblock lang:javascript %}
//in operator
var object = {
    count: 0;
    related: null;
};

if ("count" in object) {
    //code
}

if ("related" in object {
    //code
}

//code won't excute
if (object["count"]) {
    //code
}

if (object["related"] != null) {
    //code
}
{% endcodeblock %}
上述例子可以看出，判断一个属性是否存在，也需要考虑到该属性值本身是假值的情况。因此把属性值本身作为判断条件，或与`null`比较，都不是好的办法。
