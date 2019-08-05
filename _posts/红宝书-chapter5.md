---
title: "JavaScript类型方法总结与函数实例方法"
date: 2017-03-22 15:08:00
cover: "6.jpg"
category: "JavaScript"
tags:
    - JavaScript
---
>关键知识点
  * 类型方法
  * `call`、`apply`和`bind`

# 引用类型
这一章有点像API手册了。
## Object
JavaScript中的对象有点像Python中的`dict`。注意不能有尾逗号，IE7之前和Opera会报错。
注意，如果给一个函数传递的参数包括若干必要参数和若干可选参数，可以结合使用传命名参数和封装参数的方法。即对于必要参数，传递命名参数；对于可选参数，封装成一个对象传递。
<!--more-->
## Array
关于这个类型，需要注意的点：
1. `length`属性不是只读的，可以修改，但会同步到实例上。比如给一个长度为4的数组设置`length = 3`会移除最后一项；设置为5会添加一个`undefined`对象到数组里。
2. 元素个数上限：4294967295个
3. 检测数组类型可以用`instanceof Array`但需要注意`Array`的构造函数可能被修改，于是可能出现此`Array`非彼`Array`的情况。
4. 最好的办法是`Array.isArray()`。
5. 栈方法`push`、`pop`，队列方法`shift`、`unshift`返回修改后的数组长度；队列方法
6. 排序方法像Java、Python一样可以接受自定义比较器。
7. 切片方法
  * `slice()`：非破坏性切片方法，返回副本；负值+数组长度转化为正值计算。
  * `splice(operation_position, delete_times, add_element)`：破坏性切片方法，改变原数组。

8. `indexOf(element, start)`和`lastIndexOf(element, start)`，一个从前往后一个从后往前；
9. 迭代方法均为非破坏性方法：
  * 判定方法：`every(function)`
  * 过滤方法：`filter(function)`
  * 迭代方法：`forEach(function)`
  * `map()`
  * `some`：类似判定方法，只是有一个返回为`true`就返回`true`。

10. 归并方法：有`map`就应该有`reduce`和`reduceRight()`，它们的区别在于从前往后和从后往前。

## Date
说起Date想到了一个通过让CPU空转实现JavaScript中的`wait`的放法，就是获取当前时间记录下来，一直检测当前时间，发现当前时间是记录时间+5000ms的时候返回。

`Date.parse()`可以接受若干种形式的字符串解析为日期对象。

## RegExp
{% codeblock lang:javascript %}
var expression = / pattern / flag；
var expression_1 = new RegExp("pattern", "flag")
{% endcodeblock %}
flag包括：
* `/g`：非贪心匹配
* `/i`：不区分大小写
* `/m`：多行匹配

需要注意的是传入构造函数的元字符需要**双重转义**，比如要想匹配`[`，需要传入`\\[`。

`RegExp`实例方法包括：
* `pattern.exec(text)`：有匹配返回匹配，类似Python里的`match`对象；无匹配返回`null`。分组信息也类似，第0项是整个匹配字符串，类似`group(0)`。

## Function
没有重载。同名函数将发生覆盖。  
函数名都是指向函数体的指针。
ES5规范了函数对象的一个属性：`caller`，它指向该函数的调用方。
一个函数拥有`arguments`对象。可以通过`arguments.callee`取到函数本身，再通过`arguments.callee.caller`取到它的调用方。但注意，严格模式下`arguments.callee`会报错。

### `call`、`apply`与`bind`
有了前面关于作用域的知识后，来讨论这些函数实例方法就容易一些了。
相同点：
1. 都是用来在特定的作用域中调用函数，相当于设置`this`的值；
2. 接受的参数内容相同、用法相同
3. 都没有返回值
不同点：
`apply`的第二个参数是数组，或者`arguments`对象；`call`除了第一个参数是指定的对象作为作用域之外，其他都是直接传的参数。

而ES5定义了一个封装函数`bind`，它能够在完成`call`和`apply`的工作的同时，创建一个新的函数实例作为返回值，并将设置的作用域绑定在该函数实例的`this`上。
{% codeblock lang:javascript %}
var objectSayColor = sayColor.bind(o);
objectSayColor();
{% endcodeblock %}

## 基本包装类型
`Boolean`、`Number`、`String`本来是基本数据类型，但它们都应该有一些符合自身类型特征的方法。基本包装类型的作用就是，如果使用它的类型的一些方法，字比如字符串的一些方法，则再使用方法时将它转换为对象实例，调用实例中的方法；执行完毕后再销毁实例对象，让它变为一个普通的基本数据值。生命周期只有一瞬间，也不能为它指定属性。

### Number类型的一些方法
* `toFixed(n)`：按四舍五入保留n位小数，返回字符串
* `toExponential(n)`：科学计数法，底保留n位小数，返回字符串
* `toPrecision(n)`：将一个数字按照合适的格式处理（小数还是科学计数法）使得总位数为n，返回字符串

### String类型方法
* `charAt(index)`
* `charCodeAt(index)` 输出字符编码
* `concat(string, str, ...)`
非破坏性切片：  
* `slice(start, end)`字符串版的切片
* `substring(start, end)`正参数与上面行为相同，负参数全转为0
* `substr(start, offset)`第一个负参数，加字符串长度；第二个转为0

位置：  
`indexOf(char, start)`和`lastIndexOf(char, start)`从前往后和从后往前的区别，可以传第二个参数标明起始搜索位置
Trim：  
`trim()`删除前后缀空格返回副本；类似的还有Chrome支持度 `trimLeft()`和`trimRight()`。
正则相关，参数可以为字符串或`RegExp`对象：  
`match(regexp)`返回匹配对象的数组
`search(regexp)`返回匹配索引，无匹配返回-1
`replace(regexp, text)`传入字符串不能全局替换，要想全局替换必须传入带`g`的正则。结果分组类似Sublimetext的$表示法。
其他还有大小写转换，`split`、`join`等，就不记了。

## 单体内置对象
关于`eval`：
* 有点内联代码的意思/C语言中`# define`的意思，解析器内的字符串，在解析时会替换成同内容的代码。
* 内部代码中所有变量和函数不会被提升，因为它们是字符串
* 严格模式下，`eval()`中的任何变量或函数都对外不可见

# `Math`对象
包括最大、最小、上下取整、四舍五入取整、随机数、绝对值、开平方、三角函数、指对幂数等等，按照英文一翻译基本都能对上。不用刻意去记。
