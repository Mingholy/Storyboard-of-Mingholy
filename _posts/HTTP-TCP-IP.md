---
title: "HTTP、TCP/UDP和IP"
date: 2017-03-24 11:35:00
cover: "2.jpg"
category: "网络"
tags:
    - 网络
---

# HTTP
HTTP协议是一种**无连接**、**无状态**的网络传输协议。
* 无连接：每次连接只处理一个请求。服务器处理请求，得到客户端应答后，即断开连接，无需更多程序，可以节省时间。
* 无状态：个人理解：没有上下文。有好处也有坏处，好处是各独立的请求应答速度快；坏处是对于存在上下文的请求，需要把上下文也传输过去，单次请求传输数据量大。

<!--more-->

## URI与URL
软件工程里曾经各种提及的URI叫“通用资源标识符”，是在一个软件系统里标记资源访问方式\位置的字符串。  
URL是一种特殊URI，在网络请求中要求具有特定的格式：
```
http://host[":"port][abs_path]
```
其中`port`默认为80端口。

## HTTP请求
三部分组成
* 请求行
* 消息报头
* 请求正文

## HTTP请求行
请求格式为：
```
[Method] [Request-URI] [HTTP-version] CRLF
```
请求方法包括：
* GET:请求URI标识的资源
* POST:在Request-URI所标识的资源后附加新的数据
* HEAD:请求获取由Request-URI所标识的资源的相应消息报头
* PUT:请求服务存储一个资源，并以Request-URI作为其标识
* DELETE:删除资源
* TRACE:远程诊断服务器
* CONNECT:用于进行代理传输，如使用SSL
* OPTIONS:询问可以执行哪些方法
它们的用途从名称上就能知道大概。

HTTP的请求行和响应行存在一些不同，因此分开总结；而请求和响应的消息报头则类似，因此后面单独一节总结。响应正文跟应用场景和业务有关，不做介绍。

## HTTP响应行
响应格式为：
```
[HTTP-Version] [Status-Code] [Reason-Phrase] CRLF
```
其中`Status-Code`就是所谓的状态码，后面的`Reason-Phrase`就是对该状态的文本描述。  
重要的HTTP响应状态码有：  
200:OK  
301:永久移动，返回新的URI，浏览器会自动定向到新的URI。  
302:临时移动，客户端应继续使用原有URI  
304:未修改，此时服务器不反回任何资源，客户端转而访问缓存  
400:客户端请求语法错误，不能识别  
401:请求要求用户身份认证  
403:禁止访问  
404:未找到资源  
500:服务器内部错误  
502:网关错误：充当网关或代理的服务器接收到了一个无效的请求  
503:超载或系统维护引起，暂时无法响应客户端请求  

## HTTP消息报头

### 通用头部
`Cache-Control`和`Pragma`:指定缓存指令；`Pragma`是HTTP1.0的缓存指令报头。可能值包括：
  * 请求缓存指令：
    * `no-chache`
    * `no-store`(据说会影响性能)
    * `max-age`
    * `max-stale`
    * `min-fresh`
    * `only-if-cached`
  * 响应缓存指令：
    * `public`
    * `private`
    * `no-cache`
    * `no-store`
    * `no-transform`
    * `must-revalidate`
    * `proxy-revalidate`
    * `max-age`
    * `s-maxage`

`Connection`:连接选项，保持连续或者通信完毕后关闭等
`Date`:表示消息产生的日期和时间
`Transfer-Encoding`
`Upgrade`
`Via`

## 请求报头
`Accept`:指明接受哪种类型的信息  
`Accept-Charset`:指明接受的字符集。未设置该域时，默认是所有字符集都接受  
`Accept-Encoding`:编码集，同上  
`Accept-Language`:自然语言，同上  
`Authorization`:权限验证报头，如果收到服务器响应401未授权，可以发送一个包含该报头域的请求要求验证。  
`Host`:指定被亲求资源的Internet主机和端口号。一般这一信息包含在了请求的URL里。  
`Connection`  
`Cookie`  
`Referer`  
`User-Agent`:该报头域包含了用户浏览器信息。

## 响应报头
响应报头中包含了一些请求报头中不能附加的信息，比如服务器的回应、对请求访问的资源下一步访问的信息。  
`Location`:用于重定向接受者到一个新的位置，比如更换域名时  
`Server`:包含服务器的一些信息，与`User-Agent`对应  

## 一次典型的HTTP通信
1. 建立TCP连接
2. 发送请求行
3. 发送消息报头
4. 服务器应答，返回响应行
5. 返回响应消息报头
6. 返回响应主体
7. 关闭连接，或根据`Connection`域中的内容，保持连接
