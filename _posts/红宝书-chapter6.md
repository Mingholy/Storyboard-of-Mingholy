---
title: "JavaScript继承"
date: 2017-03-22 17:00:00
cover: "7.jpg"
category: "JavaScript"
tags:
    - JavaScript
---
>关键知识点
  * 继承的各种方式

关于面向对象，已经在[IFE2017-105的笔记](http://ming-holy.space/2017/03/13/IFE2017-105/#数据持久化与面向对象)中总结了一些。这里总结一下关于继承的问题。
# 继承
书中第一段提到了，一般的OO语言支持两种继承方式，一种是接口继承，另一种是实现继承。
* 接口继承：Java中的接口继承就是只声明方法，但不实现，由实现接口的类来进行具体实现。这样实现类就继承了接口类，同一个接口可以在不同的地方存在不同的实现，这也就实现了多态。
* 实现继承：父类中定义并实现好方法，然后子类在继承父类的同时也就继承了该方法的实现，可以直接拿来用。
JavaScript是没有函数签名的，像接口继承那种只继承方法特点，比如参数、名字等，就不能实现。只能通过原型链模式支持实现继承。
<!-- more -->

## 原型链
首先复习一下原型。
构造函数和其原型对象之间，一定会存在一对互相指向对方的指针。其中原型对象中的`constructor`属性存储了指向构造函数的指针，而构造函数中的`prototype`指向了原型对象。
那么假如现在我们有两对原型对象与构造函数，命名为：
1. `子类构造函数`和`子类的原型对象`
2. `父类构造函数`和`父类的原型对象`

我们用父类构造函数来实例化一个对象并把它赋值给子类的原型对象，即：
{% codeblock lang:javascript %}
SubType.prototype = new SuperType();
{% endcodeblock %}
注意此处的等号。这里的方式是，直接用父类__实例__替换（重写）子类的__原型对象__。父类的实例里一定有一个`prototype`属性指向`父类的原型对象`。现在子类的原型对象里就存在该指向父类原型对象的属性了。接着通过`子类构造函数`实例化出来的子类对象就会都拥有父类的属性和方法，包括：
1. 构造函数的指向：和它原型中的`constructor`指向一致。而因为它的原型是父类的实例，父类实例的`constructor`继承自它的原型对象，而它的原型对象是`父类原型对象`，于是就和`父类原型对象`中`constructor`的指向一致，
2. 原型对象的指向：一个父类的实例

于是一个原型链就建立起来了。同作用域链非常类似，当我们想访问一个对象的属性时，先从对象本身的属性开始搜索，如果没有就寻找它原型的属性，一级一级向上查找，直到找到`Object.prototype`。

原型链继承有两个问题：
1. 毕竟子类的原型是父类的一个实例，假如该实例继承了父类的某个引用类型值，则子类的每个实例对象都会共享该引用类型值，改一处所有实例的都被改。
2. 不能向父类构造函数传递参数（书中原话：__没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。__）

关于第二个问题可以参考：[javascript 里子类型的构造函数的实例为什么不能向父类型的构造函数实例传递参数？](https://segmentfault.com/q/1010000008512224?_ea=1678065)

## 借用构造函数
上面的参考文献中，有个回答是这样的：
{% codeblock lang:javascript %}
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
}

function Man(name,age,job,mustache){
    this.mustache = mustache;
    // 改成这句话
    Person.call(this, name, age, job)
    Man.prototype = new Person();
}

var m = new Man('Anthony',27,'PE');
m.name;// Anthony
{% endcodeblock %}
实际上是以子类的新实例作为作用域调用父类的构造函数，这样父类的构造函数中的`this`就指向了该新实例，各种属性也就是新实例自己拥有的副本，而非类共享属性。
1. 需要注意的是：这时这些属性实际上是实例对象的属性，其实和类继承没啥太大关系了，就好像我`new`了一个`Object`然后手动给它添加各种属性，而把它当做一个类的对象看待，因为这个类的所有实例都有相同的属性
2. 所有的属性和方法，都是通过构造函数挂载到对象上去的，父类构造函数中的方法，子类是调用不了的，因为连它的属性__事实上__都不是继承来的，更不用提方法了。除非在子类构造函数里添加方法属性。这就有另外一个问题了：复用无从谈起。每个实例都有这些一模一样的函数副本。

## 组合继承
和创建对象的方法类似，这里也有组合构造函数与原型的方法。通过原型链实现原型属性和方法的继承，通过构造函数实现实例属性的继承。  
{% codeblock lang:javascript %}
function SuperType(name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

//为父类原型添加方法
SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {

    //继承属性，其中name是原型属性
    SuperType.call(this, name);

    //子类实例属性
    this.age = age;
}

//继承方法
SubType.prototype = new SuperType();

//修改构造函数指向
SubType.prototype.constructor = SubType;

//重写原型之后为子类原型添加原型方法
SubType.prototype.sayAge = function() {
    alert(this.age);
};
{% endcodeblock %}
实际上，如果没有上面修改构造函数指向的代码，也能够创建出可用`instanceof`正确验证的对象。因为在调用`new SubType(somename, someage);`时，其实和当时`SubType.prototype`的构造函数没什么关系，`new`关键字的作用可以展开成：
{% codeblock lang:javascript %}
new SubType("nico", 29) = {
    var obj = {};
    obj.__proto__ = SubType.prototype;

    /*
     * 直接通过函数名称调用子类的构造函数
     * 子类的构造函数中又调用了父类的构造函数，并把name传给它
     */
    var result = SubType.call(obj, "name", "age");
    return typeof result === 'Object'? result : obj;
}
{% endcodeblock %}
可以看到`new`关键字在寻找构造函数时并不是通过`prototype`中的`constructor`寻找，而是直接通过函数名。
但是如果注释掉，问题在于：使用`new SubType()`实例化出来的子类对象，其构造函数会变成`SuperType`。这样就造成了混乱，因此不能少了那句。

## 原型式继承
刚刚在搜索`new`关键字是否真的调用`prototype`指向的`constructor`时，发现infoQ上的一篇文章，力劝大家停止使用`new`关键字转而使用Crockford提出的`Object.create()`。本节中提到的方法，实际上也是[原型链](http://localhost:4000/2017/03/22/%E7%BA%A2%E5%AE%9D%E4%B9%A6-chapter6/#原型链)这一节的[参考文献](https://segmentfault.com/q/1010000008512224?_ea=1678065)中有回答提到过的。

首先，Crockford给出了一个函数：
{% codeblock lang:javascript %}
function object(o) {
    function F(){};
    F.prototype = o;
    return new F();
}
{% endcodeblock %}
先构造了一个空函数，将此空函数作为构造函数，形式上建立了一个自定义类型，然后将该自定义类型的原型指定为`o`，返回这个自定义类型的实例。
>从本质上讲，`object()`对传入其中的对象执行了一次浅复制。

这句话应该怎么理解？  
首先，JavaScript没有引用传参。在函数中`F.prototype = o;`这条语句实际上就把传进来的对象作为`F.prototype`的值了。也就是说，`F.prototype`指向了传进来的这个对象。这里的浅复制是指，调用`object(person)`后，`person`、`object()`函数里的`o`、返回对象`F`的原型对象，都指向了同一个对象。如果我们这时更改`person`的属性值，已经返回的对象里相应的属性值也会更改。例子：
{% codeblock lang:javascript %}
var person = {
    name: "Nicholas"
}
var instance = object(person);
person.name = "Greg";
console.log(instance.name); //"Greg"
{% endcodeblock %}
出现这种情况的两个原因：
1. `object(o)`函数对`o`进行了一次浅复制，函数外部对`person`的修改实际上就是对`F.prototype`的修改。
2. 原型具有动态性，原型对象一旦被修改，其实例也会发生变化。

得到了实例对象后，就可以对实例对象的属性值进行修改了，比如此例中的`name`属性， 基本数据类型会直接复制得到副本，不会影响其他对象。但是如果`person`中定义了一个引用类型属性，比如`friends: ["Tom", "Jerry", "Katie"]`，如果对实例对象调用方法修改它的`friends`属性，比如添加一个元素，那么`person`的`friends`属性同样会被修改。正是因为浅复制，对象`o`中的引用类型属性的指针被复制给了`F.prototype`，而通过该原型对象实例化出的所有实例也都带有该指针。所以修改的就是所有实例共享的这个引用变量。

ES5新增了`Object.create()`方法，作为规范化的原型式继承方法。Crockford的思想是，要想实现继承，必须现有一个对象作为新实例的基础。把现有的这个对象传给`object`函数实例化一个新对象，再对新对象做一些修改即可。`Object.create()`方法很好地贯彻了这个思想。它的第一个参数接受一个已有的对象，返回一个以该对象为原型的实例；它的第二个参数接受一个对象，作为对新实例对象的修改。
{% codeblock lang:javascript %}
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
}
var anotherPerson = Object.create(person, {
    name: {
        value: "Greg"
    }
})
{% endcodeblock %}
该参数将覆盖原型对象上的同名属性。
>最重要的一点：包含引用类型值的属性始终都会共享相应的值。

参考文献中力劝大家抛弃其他使用构造函数的继承方法，转而使用`Object.create`。

首先，Crockford给出的`object`函数就作为了`Object.create()`的一个Polyfill方案，既然有必要为它制定Polyfill方案，那它肯定是大有用处的。

其次，`new`关键字有不足之处：
1. 构造函数也是函数，如果一时疏忽忘记使用`new`就会造成很多麻烦
2. `new`有悖JavaScript作为原型式语言设计的初衷，完全是当年为了抱Java大腿让更多的人来用而做出的妥协

最后文章提出了两种方案来实现JavaScript式的继承，一种是创建一个基类`Class`，并提供两个接口`Class.extend`和`Class.create`分别用来继承和实例化；另一种是在基本对象上绑定实例化方法，利用`Object.create()`来实现继承。这里就不在赘述了。

## 寄生式继承
回忆一下创建对象中的寄生构造函数方法。实际上就是封装创建对象的语句。在一个函数中实例化一个特定类型的对象，然后对该对象进行增加方法、属性等操作，最后返回这个对象。

寄生式继承也类似，它只封装继承代码：
1. 使用`object`函数创建一个基于某个对象的实例
2. 给这个实例增加属性、方法
3. 返回这个对象

这个对象新增的属性、方法都由它私有。这种模式主要考虑创建符合要求的对象，而非自定义类型或构造函数。它的问题是：
1. 没有自定义类型
2. 实例方法不能复用

实际上所有返回一个符合要求的对象的方法都属于此模式，甚至连`object`函数都不是必需的。然而我觉得这可能是假的继承。

## 寄生组合式继承
组合式继承存在一个问题：
在建立好`SuperType`和`SubType`的构造函数之后，我们需要把`SubType`的原型设置为`SuperType`的一个实例，这时需要调用一次`SuperType`的构造函数；
在实例化`SubType`时，`SubType`的构造函数中又调用了一次`SuperType`的构造函数。于是两次调用`SuperType`的构造函数得到了两组`SuperType`的属性，分别位于`SubType`的原型和实例上。

寄生组合式继承的最终目的仍然是让子类型的原型是父类的一个实例，本质上是让子类的原型的原型是父类的原型。为了实现这个目的，我们可以像组合式继承那样，调用父类的构造函数得到一个实例，并把该实例作为子类的原型；我们也可以直接通过`oject`函数创建一个继承自父类原型的对象（也就是说不使用构造函数得到了一个父类实例），并把它作为子类的原型。
{% codeblock %}
function inheritPrototype(subType, superType) {
    var prototype = object(superType.prototype);
    prototype.constructor = subType;
    subType.prototype = prototype;
}
{% endcodeblock %}
它实际上是组合式继承的一个改进版，改进之处在于只调用了一次父类的构造函数，并实现了同样的功能。

## 总结
JavaScript的继承模式可谓花样繁多，里面还有很多奇技淫巧。分别用一句话来总结各种继承方式：
* 总体思路：JavaScript的继承基本上是基于__令子类原型是一个父类实例__这一思路
* 原型链：最简单地贯彻了上述思路，但父类实例中若有引用类型数据会被所有实例共享
* 借用构造函数：子类构造函数中调用父类构造函数，实现父类中引用类型对象（包括方法）的复制分发，但依赖构造函数、复用性低
* 组合继承：引用类型属性依靠借用构造函数法继承，重写子类原型后再为子类原型添加类方法，较为常用，调用了两次父类构造函数
* 原型式继承：给出`object`函数，不调用构造函数的继承，借助一个已有对象获得以该对象为原型的新对象，但引用类型属性仍会被共享，未调用构造函数不能判断自定义类型
* 寄生式继承：未调用构造函数，不能判断自定义类型，但可以通过寄生函数增强对象，产生一批具有相同方法、属性的对象
* 寄生组合式继承：最小限度地调用构造函数（一次），可判断自定义类型，引用类型属性实例所有，方法类型所有，很赞
