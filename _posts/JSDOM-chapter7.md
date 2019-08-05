---
title: "JavaScript DOM 编程艺术 笔记-第七章"
date: 2017-03-17 15:46:00
cover: "2.jpg"
category: "JavaScript"
tags:
    - JavaScript
---
# 动态创建标记
>很多内容都是关于曾经使用过的创建元素的方法，简单总结一下。

## 传统方法
传统的插入元素的方法：
1. `document.write()`插入raw HTML代码
2. 为元素设置`innerHTML`属性；它并不是W3C标准中DOM的组成部分，但是HTML5支持它。

<!--more-->
需要注意的是，`innerHTML`属性能够获取或修改的值，就是一个元素节点内部拥有的所有内容。比如有这样一个元素：
{% codeblock lang:HTML %}
<div id="testdiv">
    <p>Some <em>content.</em></p>
</div>
{% endcodeblock %}
DOM是树状的结构，他会有一个根节点是`div`，然后有子节点`p`，而`p`又有子节点`em`这样。但`innerHTML`属性值只是`<p>Some <em>content.</em></p>`。它并不关心里面的结构。

并且，`innerHTML`属性是HTML的专有属性，在呈现XHTML文档时，该属性会被忽略。

## DOM方法
具体来说就是，创建和插入方法。
创建方法是：
1. `createElement(nodeName)`
2. `createTextNode(text)`

插入方法是：
1. `appenChild(node)`
2. `insertBefore(newElement, targetElement)`

如果有需要，还可以利用已有的DOM方法，编写一个`insertAfter()`方法。
{% codeblock lang:javascript %}
function insertAfter(newElement, targetElement) {
    var parent = targetElement.parentNode;
    if(parent.lastChild == targetElement) {
        parent.appendChild(newElement);
    } else {
        parent.insertBefore(newElement, targetElement.nextSibling);
    }
}
{% endcodeblock %}
总体思路就是判断目标元素是否是它的父元素的最后一个子元素，如果是则使用`appenChild`追加新节点；如果不是则在它的下一个兄弟元素前添加新节点。
**注意：这个方法使用到的DOM方法和属性有：**
1. `parentNode`属性
2. `lastChild`属性
3. `nextSibling`属性
4. `appenChild`方法
5. `insertBefore`方法

在使用该函数前，要判断这些属性和方法是否存在。

## Ajax
虽然我很想读成荷兰的那支球队的读音，阿贾克斯，但是它的的确确就是读成[ˈeɪˌdʒæks]。
全称是"异步JS和XML：Asynchronous Javascript and XML"。  
### 简单例子
这里实现了一个很简单的例子，用来在页面文档加载完毕后，异步地通过`XMLHttpRequest`对象来发起HTTP请求，通过`GET`请求类型，来访问“服务器”上的一个文件。
{% codeblock lang:javascript %}
function getNewContent() {
    var request = getHTTPObject();
    if (request) {
       request.open( "GET", "example.txt", true) ;
       request.onreadystatechange = function () {
           if (request.readyState === 4) {
               var para = document.createElement("p");
               var txt = document.createTextNode(request.responseText);
               para.appendChild(txt);
               document.getElementById('new').appendChild(para);
           }
       };
       request.send(null);
    } else {
        alert('Sorry, your browser doesn\'t support XMLHttpRequest');
    }
}
{% endcodeblock %}
其中作为`XMLHttpRequest`对象的`request`拥有若干属性和方法。其中`open`方法用来确定发起请求的类型、访问的文件名和确定是否使用异步工作模式。而`onreadystatechange`属性则指向一个事件处理函数，当`XMLHttpRequest`对象返回响应时运行。  
`XMLHttpRequest`对象的属性`readyState`值代表请求状态：
* '0': 未初始化
* '1': 正在加载
* '2': 加载完毕
* '3': 正在交互
* '4': 完成

而代码中`onreadystatechange`则是在状态码发生变化时触发的方法。

>注意：同源策略。Ajax请求只能访问与当前HTML处于同一个域种的数据。不同协议、不同域名、不同端口都算跨域。

### 渐进增强与平稳退化
对于Ajax同样存在着当JavaScript被禁用时，如何保证可用性的问题。我们来看一下区别：
* 使用Ajax: 用户的表单将异步发送至服务器进行处理，处理结果将马上返回——比如登录失败，则只更新登录表单部分，无需刷新整个页面；显得更快、更敏捷
* 不使用Ajax: 用户的表单发送至服务器处理，处理结果返回，重新渲染页面显示处理信息，再登录、再返回......

实际上`XMLHttpRequest`只是充当一个中间人的角色，有了它页面的部分内容发出的请求就可以得到处理，否则这一部分的请求就将作为整个页面的请求发送给服务器，服务器返回的结果也将作为整个页面的更新重新渲染。当然会慢一点，但为了保证可用性，应该考虑如果不使用Ajax该如何做。
