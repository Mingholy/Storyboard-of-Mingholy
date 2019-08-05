---
title: "JavaScript DOM 编程艺术 笔记-第五章"
date: 2017-02-23 21:47:13
cover: "4.jpg"
category: "JavaScript"
tags:
    - JavaScript
---
---
## 第五章：最佳实践
### 平稳退化/优雅降级（graceful degradation)
>**平稳退化**是指虽然某些功能无法使用，但是最基本的操作仍然能顺利完成。

<!-- more -->

这些无法使用的功能就包括Javascript，因为有些浏览器不支持，或者由于用户的原因被禁用了。  
以下这两种方式，都不满足平稳退化：
1. "javascript:"伪协议;
```
<a href="javascript:popUp('http://www.example.com/');">Example</a>
```
2. 内嵌事件处理函数：
```
<a href="#" onclick="javascript:popUp('http://www.example.com/');return false;">Example</a>
```

因为当在JavaScript不能使用的时候，它们都是无效链接，点击不会产生任何作用。

但是下面的例子是可用的：
```
<a href="www.example.com" onclick="popUp('http://www.example.com'); return false;">Example</a>
```
最简洁的办法是：
```
<a href="www.example.com" onclick="popUp(this.href); return false;">Example</a>
```
这样即使JavaScript不可用，这个链接也是有效的。

### 渐进增强
其实就是说，对于元素的样式，应该放在单独的CSS文件中，使用`<link>`标签引入，而不是和HTML元素混杂在一起，这样有利于修改样式，也提高代码的可读性。

### 分离JavaScript
现在有一个需求，就是我们想对所有的`<a>`标签的`onclick`事件都作以处理，每当点击该标签时，都会触发事件处理函数。**然而**我们并不想对每个`<a>`标签都添加一个`onclick`属性。这时就应该用到元素的`event`属性了。

如果将某事件绑定在特定元素上，可以用ID选择器：

{% codeblock lang:js %}
document.getElementById(id).event = action;
{% endcodeblock %}

如果涉及到一类标签，那么就要：
  1. 遍历这些标签，判断它应该调用哪个事件处理函数
  2. 给相应的标签添加时间处理函数
  3. 阻止默认行为、冒泡等

明白了要做什么，就做吧...代码我就抄上来了：

{% codeblock lang:js %}
var links = document.getElementsByTagName("a");
for (i in links) {
    if (links[i].getAttribute("class") == popup) {
        links[i].onclick = function() {
            popUp(this.getAttribute("href"));
            return false;    
        }
    }
}
{% endcodeblock %}

### 注意文档资源载入
首先上面的JavaScript代码是操作DOM的。然而JavaScript代码的载入一般而言有两种情况，一是在`<head>`标签里，二是在`<body>`标签的尾部。通常考虑到性能和安全等因素，我们放在尾部来载入。即使如此，在载入这个脚本时，文档也不一定是完整的，DOM方法也就不一定都能用。所以需要在所有文档载入完毕之后，再来执行脚本。这就要用到`window.onload`事件了。

我们希望在文档全部加载完毕之后，自动给所有相关标签添加事件处理函数。

{% codeblock lang:js %}
window.onload = function() {
    var links = document.getElementsByTagName("a");
    for (i in links) {
        if (links[i].getAttribute("class") == popup) {
            links[i].onclick = function() {
                popUp(this.getAttribute("href"));
                return false;    
            }
        }
    }
}

function popUp(winURL) {
    window.open(winURL, "popup", "width=320, height=480");
}
{% endcodeblock %}

### 性能考虑
三个原则：
1. 减少标记与访问DOM的次数
2. 合并脚本
3. 压缩脚本
