---
title: "一道关于this的小考题"
date: 2017-08-11
cover: "4.jpg"
category: "JavaScript"
tags:
    - this
    - 作用域
    - 运算符
---
>如下代码分别得到什么结果？
```javascript
var value = 1;
var foo = {
  value: 2,
  bar: function() {
    return this.value;
  }
}

console.log(foo.bar());
console.log((foo.bar)());
console.log((foo.bar = foo.bar)());
console.log((false || foo.bar)());
console.log((foo.bar, foo.bar)());
```
答案分为两种，一种是浏览器环境中，一种是node环境中。  
浏览器环境中的结果是：
```
2
2
1
1
1
```
node环境中的结果是
```
2
2
undefined
undefined
undefined
```
如果在浏览器中将代码修改为：
```
var value = 1;
var foo = {
  value: 2,
  bar: function() {
    "use strict"
    return this.value;
  }
}

console.log(foo.bar());
console.log((foo.bar)());
console.log((foo.bar = foo.bar)());
console.log((false || foo.bar)());
console.log((foo.bar, foo.bar)());
```
则结果将变为：输出前两个2之后，报错`Uncaught TypeError: Cannot read property 'value' of undefined.`。  

## 分析
1. 非严格模式下浏览器端的`this`是默认绑定`window`对象的。
2. 严格模式下`this`不绑定目标。
3. 赋值操作符返回后者。
4. `,`返回最后一个操作数。
