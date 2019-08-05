---
title: "Ajax、Comet、Fetch与跨域"
date: 2017-03-24 10:39:00
cover: "7.jpg"
category: "前端"
tags:
    - JavaScript
    - 跨域
    - 异步编程
---

>Ajax、跨域是学习前端过程中的一个痛点，也是我实际做东西的时候遇到的一个比较棘手的问题。

# Ajax
实际上是借助内建的XMLHttpRequest（XHR）对象实现异步请求。
[JavaScriptDOM第七章笔记](http://ming-holy.space/2017/03/17/JSDOM-chapter7/)中记录了一个小例子，这里详细解读一下。

<!--more-->
## XHR对象
这里需要注意的是IE的解法。IE对XHR的实现是通过MSXML库中的一个ActiveX对象实现的。所以我们在编写带有兼容性的代码时会有这样的情况：
{% codeblock lang:javascript %}
function getHTTPObject() {
    //检测是否存在XHR内置对象，如果没有就从MSXML库里找ActiveX的实现
    if(typeof XMLHttpRequest == "undefined") {
        //注意这里建立的是全局对象
        XMLHttpRequest = funciton() {
            try {
                return new ActiveXObject("Msxml2.XMLHTTP.6.0");
            } catch (e) {}
            try {
                return new ActiveXObject("Msxml2.XMLHTTP.3.0");
            } catch (e) {}
            try {
                return new ActiveXObject("Msxml2.XMLHTTP");
            } catch (e) {}
            return false;
        }
    }
    return new XMLHttpRequest();
}
{% endcodeblock %}
所以IE早期版本（7之前）与其他浏览器的不同之处在于对XHR对象的实现方式。

### 使用XHR
使用XHR分为四个步骤：
1. 实例化，可以用上面的那个函数
2. 准备并发送请求，使用`open`
  ```
  xhr.open("get", "example.txt", false);
  ```
  最后一个参数代表是否是异步请求。注意：
  * URL是相对路径，也可使用绝对路径
  * `open`是准备请求，调用它并不会立即发送
3.监听对象状态并发送请求
由于上面的`open`方法确定了是同步请求，则它准备完毕后才能调用`send`发送请求。如果是异步请求，则需要一直判断请求的状态，以决定下一步该怎么做。所以在发送之前应该首先给监听器绑定一个处理方法：
{% codeblock lang:javascript %}
xhr.onreadystatechange = function(){
    if (xhr.readystate === 4) {
        //当这个请求完成时要做的处理

}
//绑定好处理函数后再发送请求
xhr.send(null);
{% endcodeblock %}
如果没有通过请求主体发送的数据，则必须传入`null`。
其中`readystate`属性代表请求本身的状态。前面的笔记也提到过：
  * '0': 未初始化
  * '1': 正在加载
  * '2': 加载完毕
  * '3': 正在交互
  * '4': 完成  
4. 获取响应
  首先看一下响应包含的内容：
  * `responseText`：响应主体，无论何种类型都会保存在这里
  * `responseXML`：如果响应内容类型是XML则数据也会包含在该属性中，否则为空
  * `status`：响应的HTTP状态码
  * `statusText`：HTTP状态说明
  比如监听响应状态为2xx/304时显示返回相应的内容：
  {% codeblock lang:javascript %}
  if (xhr.status >= 200 && xhr.status < 300 || xhr.status == 304) {
      console.log(xhr.responseText);
  }
  {% endcodeblock %}
  最后可以使用`xhr.abort()`取消异步请求。

### 设置XHR头部信息
发送XHR请求的同时在HTTP请求中附加的头部信息有：
* Accept：浏览器能够处理的内容类型
* Accept-Charset：浏览器能够显示的字符集
* Accept-Encoding：浏览器能够处理的压缩编码
* Accept-Language：浏览器当前语言
* Connection：浏览器与服务器间连接类型
* Cookie：当前页面的任何Cookie
* Host：发出请求页面所在域
* Referer：发出请求页面的URI
* User-Agent：浏览器代理字符串（浏览器嗅探用）

可以用`setRequestHeader()`设置请求头部信息。必须在`open`之后，`send`之前。
各请求就不详述了，`GET`请求读，`POST`请求写。

# XHR level2

## FormData
可以实例化一个`FormData`对象来存储表单。
{% codeblock lang:javascript %}
var data = new FormData();
data.append("key", "value");
//或者直接将表单元素填入构造函数
var data1 = new FormData(document.forms[0]);
//最后发送它
xhr.send(data1)
{% endcodeblock %}

## Timeout
可以设置一个响应时限（patient？），来处理超时的情况。
{% codeblock lang:javascript %}
xhr.timeout = 1000;
xhr.ontimeout = function() {
    aledrt("Some alert!");
}
xhr.send(null);
{% endcodeblock %}
记住：事件绑定要在发送之前！

## `OverideMimeType()`方法
要修改MIME类型的注意了！你的`<video>`标签从本地导入视频后遇到的两个问题之一这里就能解决了！
默认的MIME类型是`text/plain`，只要类型不是`XML`，XHR对象里`responseXML`属性就是空。可以通过修改MIME类型来将相应当做XML。
{% codeblock lang:javascript %}
xhr.open("get", "text.xml", true);
xhr.overrideMimeType("text/xml");
xhr.send(null);
{% endcodeblock %}

## 进度事件
看到有面试题说，要求用Ajax写一个文件上传组件，要求显示上传进度。这里可能就是关键。
XHR有6个进度事件：
* `loadstart`：开始接收相应的第一个字节触发
  `readystatechange`事件的替代，无需在`readystatechange`属性上绑定函数来监视`readystate`属性是否等于4，而直接用：
  {% codeblock lang:javascript %}
  xhr.onload = function() {
      //注意：status属性还是要判断的，因为返回一个失败的响应也算响应
      if (xhr.status >= 200 && xhr.status < 300 || xhr.status == 304) {
          //pass
      } else {
          //pass
      }
  }
  {% endcodeblock %}
* `progress`：接收过程中不断触发
  XHR的`onprogress`属性可以给`progress`事件绑定一个事件处理函数，它将接收到一个`event`对象，该对象属性包括：
    * `target`：指向XHR对象本身
    * `lengthComputable`：进度信息是否可用，Boolean
    * `position`：当前进度（已接收字节数）
    * `totalSize`：根据`Content-Length`响应头部确定的总预期字节数
    具体进度指示器，抄书：
    {% codeblock lang:javascript %}
    var xhr = createXHR();
    //先绑定完成响应
    xhr.onload = function(event) {
        if ((xhr.status >=200 && xhr.status <300) || xhr.status == 304) {
            console.log(xhr.responseText);
        } else {
            alert("Request was unsuccessfule: " + xhr.status);
        }
    };
    //先绑定进度监控
    xhr.onprogress = function(event) {
        var divStatus = document.getElementById("status");
        //判断进度信息可用
        if (event.lengthComputable) {
            divStatus.innerHTML = "Received " + event.position + " of " + event.totalSize + " bytes";                    
        }
    };

    //准备请求
    xhr.open("get", "altevents.php", ture);
    //发送请求
    xhr.send(null);
    {% endcodeblock %}
* `error`：请求发生错误时触发
* `abort`：调用`abort()`方法终止连接时触发
* `load`：接收到完整的相应数据时触发
* `loadend`：通信完成或者触发`error`、`abort`或者`load`事件后触发。


# **跨域！**
最难受的一个问题到来了。

## 什么是跨域？
首先，XHR只能访问与包含它的页面位于同一个域中的资源。那么怎么算是跨域呢？
1. 不同域名，包括二级域名
2. 同一域名，不同协议
3. 同一域名，不同端口
对不同域之间进行数据传输或通信的一套令行禁止规则，叫做同源策略，这是浏览器都有的一种安全功能。

受制于这种策略，一个域内的JS不能操作其他域上的对象。也就是说不同域间的通信受到了限制。但是在使用`iframe`或Ajax时，一般情况下都是跨域的，怎么办呢？

如何解决跨域问题？方法有很多种，大概分两大类：
1. 单向跨域：用于获取
2. 双向跨域：用于交互

书中的XDomainRequest是IE8、9的解决方式，MDN上显示现在已经被废弃。其他浏览器的跨域XHR对象，会有一些限制：
1. 不能使用`setRequestHeader()`设置自定义头部
2. 不能发送和接收`cookie`
3. 调用`getAllResponseHeaders()`方法总会返回空字符串

>小tip：
  无论同源请求还是跨域请求都适应相同接口，因此对于本地资源，**最好使用相对URL**，在访问远程资源时再使用绝对URL。目的是消除歧义，避免出现限制访问头部或本地cookie信息等问题。

## 单向跨域
### 方法一：JSONP
JSONP即JSON with pading，填充式JSON。
一个典型的JSONP请求：
```
http://freegeoip.net/json/?callback=handleResponse
```
可以看到该URL中含有一个参数`callback`，值应该是一个回调函数的名字。

首先应该知道的是，动态的`<script>`和`<img>`都可以不受限制地从其他域加载资源（这也就回答了我心中的疑问：通过URL加载其他站点上的图片也算跨域请求，为何能够成功？）。

其次上面指定的那个回调函数，在请求得到响应后会被执行，响应传回来的数据会自动作为参数传入到该回调函数中，我们就可以进行一些处理了。

那么最后一个问题来了，如何发送请求呢？考虑第一条，动态的`<script>`可以不受限制地从其他域加载资源，比如引用google的某些js库，CDN上的jQuery等等；我们同样可以通过创建一个`<script>`，发出加载跨域脚本的请求。而该跨域脚本实际上就是向另外一个域发出一个请求，然后获得一个json并把它传给我们定义好的回调函数。于是这个例子就可以实现了：
{% codeblock lang:javascript %}
function handleResponse(response) {
    console.log("You're at IP address " + response.ip + ", which is in " + response.city + ", " + response.region_name);
}

//动态创建脚本
var script = document.createElement("script");
script.src = "http://freegeoip.net/json/?callback=handleResponse";

//插入脚本
document.body.insertBefore(script, document.body.firstChild);
{% endcodeblock %}
当然这种方式也有问题：
1. 只支持`GET`请求，不支持`POST`等
2. 从其他域加载来的东西，可能夹杂恶意代码，这种情况作为请求发起方是没办法控制的。除非放弃JSONP调用。因此要求通信双方都是可信的。
3. 请求状态难以检测，我们除了能创建一个计时器来检测请求是否超时之外，能做的并不多。并且设置计时器也不妥当，因为获取相应所需的时间可能因人因网而异。

### 方法二：`window.name`
原理是利用了`window.name`的不变性。如果`window.location`改变了，刷新后`window.name`也会保持不变。具体步骤是：
1. 在页面A中`iframe`加载B，现在要跨域获取B的数据，就先在B中把数据赋值给B的`window.name`
2. 在A中将B的`window`对象地址改成与A同域的一个地址，这样就可以获取`window.name`了。即使此时`iframe`刷新，原来的B不在了，也可以获得之前它传来的值。

### 方法三：服务器代理

## 双向跨域
主要是一个页面包含一个`iframe`元素的情况。也有在两个页面之间传递消息的方法。

### 方法一：`document.domain`
简单来说就是，既然不同域名下不能通信，那么就手动设置对象的域名为相同的即可。
比如把主页面和`iframe`中的`document.domain`属性都设为同一域名：
{% codeblock lang:javascript %}
document.domain = "a.com";
{% endcodeblock %}
缺点是：
1. 所有子页面对象都要设置成相同域名
2. 如果一个页面上有安全漏洞，所有的页面都会compromise

### 方法二：`location.hash`
父子页面都建立一个轮询，监听自己的`location.hash`属性。
1. 发送消息通过修改对方的`location.hash`属性
2. 接收消息通过监听自己的`location.hash`属性

### 方法三：HTML5 `postMessage()`方法
`postMessage()`方法是跨文档消息传送(cross-document messaging, XDM)的核心。它可以向当前页面的`iframe`元素或弹出窗口传递消息。
{% codeblock lang:javascript %}
var iframeWindow = document.getElementById("myframe").contentWindow;
iframeWindow.postMessage("A secret", "http://www.wrox.com");
//即
contentWindow.postMessage（messageString, targetDomainName;
{% endcodeblock %}
只有当指定框架中的文档来源于目标域时，消息才会传递到目标框架中；否则不会。

>注意：`contentWindow`是一个`window`对象，它可以是一个`iframe`也可以是一个新的页面窗口。

该方法传送的消息，会触发`window`对象的`message`事件，但是这个事件是异步触发的，因此发消息与接收消息之间存在延迟。`message`事件的处理程序`onmessage`包括：‘
* `data`：消息字符串
* `origin`：发送方所在域
* `source`：发送消息的文档的`window`对象的代理。用于在上一条消息的发送方窗口中调用`postMessage()`方法，发消息验证发送成功。
{% codeblock lang:javascript %}
EventUtil.addHandler(window, "message", function(event) {

    //检测发送方的域是指定已知的域
    if(event.origin === "http://www.wrox.com") {
        processMessage(event.data);

        //发送确认回执
        event.source.postMessage("Received!", "http://p2p.wrox.com");
    }

}
{% endcodeblock %}

## 其他方法

### Preflighted Requests


### 跨浏览器的CORS
