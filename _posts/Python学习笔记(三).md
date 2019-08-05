---
title: "Python学习笔记(三)-关于面向对象"
date: 2017-03-03 15:21:00
cover: '6.jpg'
category: "Python"
tags:
    - Python
---
## 面向对象
要给姚老师用写一个医案处理的脚本，想用Python来实现。医案相对来说是很结构化的。不同字段下的文字有不同的特征。觉得用面向对象的方法能够更好地处理这一块的问题。所以再看了一下Python的面向对象知识。  

<!-- more -->

### 类与实例
其实这一部分的内容跟Java非常类似。Python中类的定义方式如下：

{% codeblock lang:python %}
class ClassName(object):
     def __init__(self, attr1, attr2):
          self.attr1 = attr1
          self.attr2 = attr2

     def print_attr1(self):
          print(attr1)
{% endcodeblock %}

声明：`class`关键字加类名，`object`指明继承自哪个父类。当然所有的类最终都会继承自`object`，它应该是所有类的父类。  
然后是构造函数。这个东西在Python里以双下划线包裹，是一种特殊的方法，并且它的第一个参数永远是`self`，该参数代表实例化的对象本身。给它传参数，可以在实例化的时候，为新建的对象绑定一些数据。然后就可以给这个类添加一些类属性和方法了。  

跟Java的`private`、`public`、`protected`这种类似的，`python`里用下划线来发挥这些关键字的作用。**私有变量**用双下划线开头，注意是开头。`init`这种特殊的方法才是两面包裹。  
私有变量只能被对象内的方法访问，外界不能访问。要想访问或者修改私有变量就要用到类似setter和getter这种方法了。概念上大概所有面向对象的语言都是相同的。怪不得姚老师说，面向对象的语言学会一种，其他的都能理解了。  
那么如果可以通过直接访问、修改对象的成员变量就能做到的事情，为什么要建立相应的方法来做呢？因为方法可以检测参数合法性，以及做到很多与安全性、健壮性相关的事情。  

### 访问限制
双下划线开头结尾的是特殊变量，或说是保留变量。特殊变量可以直接访问，但是我们不能新建这样的变量。  
在廖雪峰老师教程的评论里边看到这样的解释：
>在一个实例里， `__girl`表示“我是贞女，你不能上我”； `_girl`表示“你虽然可以上我，但你应该把我看做贞女”；`girl`表示“我是荡妇，谁都可以上我” 但是Python仍然可以用`_实例名__girl`强上贞女

（越污记得越牢）
后面的补充例子意思是：
比如有一个类`Student`，类成员变量有`__name`和`__score`，它们都是私有变量，只能通过`setter`和`getter`修改和访问。它有一个实例对象：`bart`，这时如果通过“取属性值 ”直接赋值来修改实例里的`__name`值：

{% codeblock lang:python %}
bart.__name = "new name"
{% endcodeblock %}

是行不通的，因为不通过setter，而通过“.”访问的变量，实际上并不是该类实例里的私有变量，而是另一个在赋值时为这个类实例创建的（临时？）变量。我们可以通过

{% codeblock lang:python %}
bart.__name
{% endcodeblock %}
得到“new name”输出，但是如果使用
{% codeblock lang:python %}
bart.get_name()
{% endcodeblock %}

得到的就不是“new name”而是之前设置的类私有变量的值。
因为该实例的私有变量已经被改写成了`_Student__name`，即这种带类名的私有变量。虽然可以通过`bart._Student__name`来访问它，但是强烈不推荐，因为不同版本的解释器的写法不同，不一定能够访问得到。

{% codeblock lang:python %}
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    def set_name(self, name):
        self.__name = name

    def get_name(self):
        return self.__name

bart = Student("bart", 98)
print("bart.get_name():" + bart.get_name()) #bart.get_name():bart
print("excute:bart.__name = 'new name'") #excute:bart.__name = 'new name'
bart.__name = 'new name'
print("bart.__name:" + bart.__name) #excute:bart.__name = 'new name'
print("bart.get_name():" + bart.get_name()) #excute:bart.__name = 'new name'
print("excute:bart._Student__name = 'new name'") #excute:bart.__name = 'new name'
bart._Student__name = 'new name'
print("bart.get_name():" + bart.get_name()) #excute:bart.__name = 'new name'
{% endcodeblock %}

### 继承与多态
人是个好奇怪的生物。我也是Java初学，但是我在写Java的时候就不觉得Java有多好，反而觉得这种强类型的语言很蛋疼的样子。但是在写Python的时候，虽然很舒服，但是同时也会觉得Java是一门非常棒的语言。这个棒是写起来爽的意思，当然这里面有IDE的很大功劳，但是这种感觉的一部分也来自于Java这门代表性的面向对象语言的很多牛逼概念。比如说继承、多态、接口、抽象类等等。  

当若干个类有相同的属性或者方法的时候，可以把它们的共同点抽象成一个父类，让这若干个类都继承自该父类，那么就得到了它们的共同点；如果它们各自还有自己的特殊属性和方法，再分别实现或者`@override`父类的方法。  

多态的大概意思是一个实例属于一个类A，该类又是另一个类的子类B，那么这个实例就既属于类A又属于类B。这样当一个函数的参数要求传入一个父类B的对象时，不论它有多少子类，它子类的对象都可以作为合法参数传给这个函数。这样调用这个函数就可以不用针对不同子类对象修改里面的操作。所以这一块有一个原则叫“开闭原则”：
1. 对扩展开放，允许新增子类。
2. 对修改封闭，不需要修改依赖父类的函数。

这里还有一个概念叫 duck typing 鸭子类型。  
>当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。

Java是静态语言，对类型的要求很严格，这也是为什么有人吐槽Java可以把一个小项目做成中型项目，把一个中型项目做成大型项目的原因。它要求各种类型都要有明确的定义和继承关系。要求传入的参数是某个类型的，那么就必须是这个类型或其子类的，否则就不能调用方法。

Python是动态语言，也没有强烈的类型要求，它只需要传入的对象有相应的方法。如果它做的事情与要求的类对象相同，那么即使它实际上并非这个类的对象，解释器都会将它视作合法的。

### 获取对象信息
判断对象类型的方法:
* `type(object)`可以判断基本数据类型，`int` `str`等。  
* `type(function) == types.FunctionType`可以判断函数类型。包括`BuiltinFunctionType`、`LambdaType`、`GeneratorType`等。  
* `isinstance(object， class)`判断类对象。可以判断基本数据类型、类对象。  
* `isinstance([1, 2, 3], (list, tuple))`判断前者是否属于后者列表中的类型之一。  

`dir()`返回对象所有属性和方法。这时还注意，`__xxx__`是类自带特殊方法。如果有特殊需求，可以重写它们。  
类对象有自带的`getattr()`和`setattr()`方法。可以取到或者设置某属性的值，还可以通过`hasattr()`判断对象是否有某属性。试图获取不存在属性的值会出现`AttributeError`错误。

### 实例属性和类属性
**类属性被所有类的实例对象共享。实例属性只为实例化的该对象所拥有。**

## "高级"面向对象...
### 使用`__slots__`
上面的内容在遇到私有成员变量的时候，曾经用到`bart.__name = "new name"`这样的语句。实际上这里并没有对该实例的成员变量`__name`做改变，因为实例里边的变量要想访问，要使用`_Student__name` 这样的形式。而这里的`__name`实际上是新建了`__name`这个变量，并且将它绑定到`bart`这个实例对象上。  

但是这样做会带来一个问题，就是我们随意创建一个变量，都可以绑定到该类的实例上去。于是同一个类的不同实例对象就可能在程序中被挂载了不同的属性。这样的混乱是应该避免的。而`__slots__`就用来避免这种情况。  
在定义类时添加`__slots__ = ('name', 'age')`可以规定，该类的对象只能被绑定`name`和`age`两个属性，而其他的属性会返回`AttributeError`错误。

需要注意的是这种限制方法只对该类本身的对象有效，而对它的子类对象无效。

{% codeblock lang:python %}
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
{% endcodeblock %}

### 使用`@property`(属性化方法)
上面提到，给类添加`setter`和`getter`方法可以在设置属性值时检查该值是否合理合法。但是需要调用对象的相关方法。那么可不可以直接使用属性，或者说看起来就像只使用属性的样子，来实现属性值访问和设置呢？Python装饰器可以给函数动态添加功能，在类中，`@property`装饰器可以把一个方法变成属性调用。而`@属性名.setter`可以指明对该属性修改的`setter`方法。如果一个属性只设置了`@property`，那么该方法就是只读方法。  
例子：

{% codeblock lang:python %}
class Student(object):

    @property
    def score(self):
        return self.__score
    @score.setter
    def score(self, score):
        self.__score = score;

    @property
    def sex(self):
        return self.__sex
{% endcodeblock %}

### 多重继承
当不同的分类方式存在类级别的重合时，比如教程中出现的，蝙蝠同属哺乳动物和会飞的动物，不同的分类标准结合时会产生大量的子类。  

采取多重继承的做法可以减少这种分类带来的麻烦，即一个子类继承一个父类的同时，还可以继承其他父类。比如：

{% codeblock lang:python %}
class Bat(Mammal, Flyable):
    pass
{% endcodeblock %}
这其中我们应该有一个概念，就是我们应该制定一个继承方式作为主要的继承链；而其他的需要同时继承的类则称为MixIn。比如上面的例子，我们以“蝙蝠是哺乳动物”这个继承关系为主要继承，而其他的`Flayable`、食肉动物`Carnivorous`，则是MixIn这种形式继承。

{% codeblock lang:python %}
class Bat(Mammal, FlyableMixIn, CarnivorousMixIn):
    pass
{% endcodeblock %}

关于Python面向对象这一部分还有定制类、枚举类和元类没有做笔记。以后再补充。
