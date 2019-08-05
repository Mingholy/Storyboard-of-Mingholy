---
title: '编写可维护的JavaScript:第二部分-I'
cover: '2.jpg'
date: 2016-03-07 20:50:24
category: JavaScript
tags:
    - JavaScript
---
>这一部分的主题是编程实践。主要的内容包括：
* UI的松耦合：即JavaScript、CSS和HTML三者之间的解耦
* 避免使用全局变量：创建全局变量可能引起的问题和避免方式

<!--more-->
## UI层的松耦合
松耦合是指，修改一个组件而不需要更改其他组件时，就做到了松耦合。单个组件的内聚度很高，只通过有限的方式同其他组件协同工作，这样既保证了单一组件的精炼，又增加了系统的稳定性。
### JavaScript$\leftrightarrow$CSS
在CSS中，可以使用`expression()`来插入内联的JavaScript代码。比如：
{% codeblock lang:css %}
.box {
    width: expression(document.body.offsetWidth + "px");
}
{% endcodeblock %}
这样做会影响性能，因为浏览器会高频率重复计算CSS，频繁调用这些表达式。而且有一个原则就是，CSS只干CSS应该干的事，有关于JavaScript的，请放到JavaScript文件中。这样假如当JavaScript出现问题的时候，我们会第一时间查看JavaScript，而非去CSS中查找隐藏的陷阱。按照作者的话说，没有JavaScript的时候也会报JavaScript的错，这真是见了鬼了。

同样虽然JavaScript时常操作DOM修改它的CSS，但是把它们捏到一起不是一个好的做法。跟上面的错误类似，这本应该是CSS应该承担的工作。包括直接给DOM赋值一整个CSS字符串，这都减低了代码的可维护性。

解决方式应该是：**增加类/减少类**。

JavaScript可以任意增删类，而类中的CSS是可以任意更改的。这样就降低了JavaScript和CSS的耦合程度。

### JavaScript$\leftrightarrow$HTML

先看一个例子：
{% codeblock lang:html %}
<button onclick="doSomething()" id="action-btn">Click Me</button>
{% endcodeblock %}
这样做有两个缺点：
1. 当按钮点击事件触发的时候，该函数`doSomething()`必须已经存在。如果该函数出现在这个HTML标签的后面或者它存在于外部JavaScript文件。无论如何只要用户点击按钮时，这个函数还不存在，那么就会报JavaScript错误。
2. 如果函数名被修改，随之而来的就是要根据调用关系修改所有调用它或者它调用的其他函数并且同时需要修改HTML标签。这明显属于不必要的任务量增加。

使用原生的JavaScript可以选择函数声明→选择器选择对象→添加对象绑定事件这种方式。
比如：
{% codeblock lang:javascript %}
$("#action-btn").on("click",doSomething);
{% endcodeblock %}
另外在HTML中嵌入JavaScript标签也不是一个好的习惯。所有的JavaScript都应该在它应该属于的外部JavaScript文件中。

同样JavaScript也不应该嵌入HTML代码。但是由于现在Web大多数是动态的，JavaScript总会向页面注入/修改/删除HTML标签，在注入标签时，明显将标签内容写死在JavaScript代码里是不好的。要做到这一点，有包括但不限于以下方法：
1. 从服务器加载
    将模板置于服务器，使用XMLHttpRequest对象来获取外部标签。示例代码：
    {% codeblock lang:javascript %}
    function loadDialog(name, oncomplete){
        var xhr = new XMLHttpRequest();
        xhr.open("get", "/js/dialog/" + name, true);

        xhr.onreadystatechange = function() {

            if (xhr.readyState == 4 && xhr.status == 200) {

                var div = document.getElementById("dlg-holder");
                div.innerHTML = xhr.responseText;
                oncomplete();

            } else {
                //code dealing with exception
            }
        };

        xhr.send(null);
    }
    {% endcodeblock %}
    或者使用jQuery：
    {% codeblock lang:javascript %}
    // jQuery
    function loadDialog(name, oncomplete) {
        $("#dlg-holder").load("/js/dialog/" + name, oncomplete);
    }
    {% endcodeblock %}
    这里的方式是，JavaScript向服务器发起一个请求，将HTML模板代码作为字符串来获取。后面的jQuery就是类库对前面的操作进行了封装，可以使load操作直接挂在到DOM元素上。当需要注入的HTML很多的时候，这种方法很实用，但是需要考虑到安全问题：从服务器获取模板的方式很容易造成XSS漏洞，需要前后端统一对模板文件的转义处理，如<、>、"等。
2. 简单的客户端模板
    客户端模板就是一些带有可变参数的HTML标签，这些标签中的某些变量可以被JavaScript访问并修改成正常的HTML代码，然后将其注入到DOM显示出来。  
    尽管含有参数，但是它终究还是HTML片段，考虑到本节的标题，我们肯定是要把它同JavaScript独立开来的。一般的做法是直接将它们放在页面上。具体的操作可以有两种方式，一是把所需要的模板作为注释放在HTML标签之间，如下：
    {% codeblock lang:html %}
    <ul id="mylist"><!--<li id="items%s"><a href="%s">%s</a></li>-->
        <!--other code-->
    </ul>
    {% endcodeblock %}
    这样我们就可以通过取`mylist`这个DOM元素的第一个子节点来获得模板。将其格式化并插入HTML的函数如下：
    {% codeblock lang:javascript %}
    function addItem(url,text) {
        var mylist = document.getElementById("mylist"),
            templateText = mylist.firstChild.nodeValue,
            result = sprintf(template, url, text);
        div.innerHTML = result;
        mylist.insertAdjacentHTML("beforeend", result);
    }

    addItem("/item/4", "Fourth item");
    {% endcodeblock %}
    另一种做法是使用带有自定义`type`属性的`<script>`标签。浏览器会将此标签识别为JavaScript代码，但我们也可以给该标签设置不同的`type`来同一般的JavaScript代码区分开来。
    比如：
    {% codeblock lang:html %}
    <script type="text/mytemplate" id="list-item">
        <li><a href="%s">%s</a></li>
    </script>
    {% endcodeblock %}
    使用该模板文本的JavaScript代码如下：
    {% codeblock lang:javascript %}
    var script = document.getElementById("list-item"),
        templateText = script.text;
    {% endcodeblock %}
    相应地，`addItem()`函数也需要进行修改：
    {% codeblock lang:javascript %}
    function addItem(url,text) {
        var mylist = document.getElementById("mylist"),
            script = document.getElementById("list-item"),
            templateText = mylist.firstChild.nodeValue,
            result = sprintf(template, url, text);
            div = document.createElement("div");

        div.innerHTML = result.replace(/^\s*/, "");
        mylist.appendChild(div.firstChild);
    }
    {% endcodeblock %}
    修改之处在于去掉了模板文本中的前导空格。这个前导空格是，在`<script>`标签中代码总是在下一行开始，如果不修改，则创建元素时会出现一个只含一个空格的文本节点。  
3. 复杂客户端模板
    个人理解是一些……模板引擎，原书中的例子是Handlebars，和简单的模板类似，也有占位符，这里的占位符使用双大括号表示。  
    比如上一节的例子：
    {% codeblock lang:html %}
    <script type="text/mytemplate" id="list-item">
        <li><a href="{{url}}">{{text}}</a></li>
    </script>
    {% endcodeblock %}
    实际上这种JavaScript模板引擎类似Ruby on Rails的erb、Node.js的ejs、jade等等，故不赘述，如果有使用需求，阅读相关的文档更有帮助。

## 避免使用全局变量
### 全局变量带来的问题
无限制地使用全局变量和全局函数，非常容易发生（基本上是一定会发生）命名冲突，这将导致变量的意外修改、覆盖等严重的错误。  
一个依赖全局变量的函数就是深耦合于上下文环境之中。如果环境发生改变，这个函数也可能就失效了。这就导致了代码鲁棒性不高。  
不仅在开发过程中会出现各种各样的错误，全局变量也可能会导致测试工作无法进行。因为需要维护两个全局环境：生产环境和测试环境。  
### 意外的全局变量
一个列子：
{% codeblock lang:javascript %}
function doSomething() {
    var count = 10;
        name = "Nicholas";
}
{% endcodeblock %}
这一段的问题有两个：一是不小心在声明变量的时候，添加了";"使得下一个声明的变量`name`变成了全局变量；二是`name`是全局对象`window`的一个属性，`window.name`属性经常用于框架（frame）和iframe场景中，修改它将影响到站点的链接打开方式。  
> 最好的规则是总是使用`var`来定义变量，即使是全局变量。这样做将降低某些场景中省略`var`所导致错误的可能性。

JSLint和JSHint总会在发现未声明的变量时报警。同样如果使用严格模式，JavaScript引擎将在这里报错。
{% codeblock lang:javascript %}
"user strict"
foo = 10;
{% endcodeblock %}
这里将会报错ReferenceError，说明为：“foo未被定义”。
### 单全局变量方式
所谓单全局变量就是使所有代码都对某**一个**全局变量有相同的依赖。依赖尽可能少的全局变量，可以在团队协作开发引入多个文件、在不同场景下使用不同文件时，保持代码的可维护性。比较著名的例子就是JavaScript的各种类库:
* YUI定义了一个`YUI`全局对象。
* jQuery定义了两个全局对象，我们所熟知的`$`和作为它的别名的`jQuery`。它仅在少数`$`失效的情况下使用。
* Dojo定义了一个`dojo`全局对象。
* Closure定义了一个`goog`全局对象。

“单全局变量”的意思是所创建的这个唯一的全局对象名是独一无二的，不会与内置API产生冲突，并且将所有功能代码都挂载在这个全局对象上。于是每个可能的全局变量都成为唯一全局对象的属性。  
#### 命名空间
即使代码中只有一个全局对象也存在全局污染的可能性。所以需要引入命名空间这一概念。  
**命名空间是简单的通过全局对象的单一属性表示的功能性分组。**  
所以我们可以创建新的对象来创建自己的命名空间：
{% codeblock lang:javascript %}
var ReadingBooks = {};

//Book 1 namespace
ReadingBooks.MaintainableJavaScript = {};

//Book 2 namespace
ReadingBooks.JavaScriptDesignPattern = {};
{% endcodeblock %}
> 一个常见的约定是每个文件中都通过创建新的全局对象来声明自己的命名空间。

如果在每个文件都需要给一个命名空间挂载东西，首先需要确定这个命名空间是已经存在的。但是我们又不能总是新建它，因为如果之前已经有人创建了这个命名空间，我们是可以将我们扩展的功能继续挂载上去的，而这也是原书作者比较推荐的做法，即“扩展原有功能”。  
下面这个函数可以非破坏性地处理命名空间，即在命名空间存在时使用原有的命名空间，如果不存在就新创建一个。
{% codeblock lang:javascript %}
var YourGlobal = {
    namespace: function(ns) {

        //format new namepsace
        var parts = ns.split("."),
            object = this,
            i, len;

        //check exitstence
        for (i=0, len=parts.length; i < len; i++) {
            if (!object[parts[i]]) {
                object[parts[i]] = {};
            }
            object = object[parts[i]];
        }

        return object;
    }
};
{% endcodeblock %}
全局对象`YourGlobal`的一个成员函数`namespace()`要求传入参数为一个字符串，用法如下：
{% codeblock lang:javascript %}
// Create a new namespace
YourGlobal.namespace("Books.MaintainableJavaScript");

// Assign a value to an attribute
YourGlobal.Books.MaintainableJavaScript.author = "Nicholas";

// Create another namespace undestructively: won't modify YourGlobal.Books, just new a object.
YourGlobal.namespace("Books.HighPerformanceJavascript");
{% endcodeblock %}
每个文件开头都可以调用该方法使它所使用的命名空间总是存在的，还不会对已有的命名空间造成破坏。这样不仅不用在每次使用前判断命名空间是否存在，同时维护性更好。
#### 模块
这一部分比较……不是很好懂。因为没有什么Java基础，所以开始看这一段的时候始终对“模块”这一概念非常模糊。这里参考了一篇文章：[前端模块化](http://www.cnblogs.com/dolphinX/p/4381855.html)，对理解这部分内容很有帮助。  
Java中有一个`package`的概念，这里实际上是在实现类似的特性。如果单纯地把实现某以特定功能的语句放在一起，很容易在命名上出现冲突，也就是我们一直在努力避免的全局变量污染。  
之前已经考虑了使用原生JavaScript添加命名空间。另一种基于单全局变量的扩充方法是使用模块（modules）。模块是一种通用的功能片段，它并没有创建新的全局变量或者命名空间，**所有的这些代码都存放于一个表示执行一个任务或发布一个接口的单函数中。**模块可以用一个名称表示，同时它也可以依赖其他模块。
1. YUI模块
    实际上YUI模块是使用YUI JavaScript类库来创建新模块，在模块内使用命名空间的方式来管理代码。  
    `YUI.add()`的参数是依次是：模块名字、工厂方法、可选的依赖列表。
    {% codeblock lang:javascript %}
    //new module
    YUI.add("module-name", function(Y) {

        //module code
    }, "version", { requires: [ "dependency1", "dependency2" ] });

    YUI.add("my-books", function(Y) {

        //add a namespace
        Y.namespace("Books.MaintainableJavaScript");

        Y.Books.MaintainableJavaScript.author = "Nicholas";
    }, "1.0.0", { requires: [ "dependency1", "dependency2" ] });
    {% endcodeblock %}
    调用`YUI().use()`传入要加载的模块名称。
    {% codeblock lang:javascript %}
    YUI().user("my-books","another-module", function(Y) {

        //code
    });
    {% endcodeblock %}
    执行顺序是，首先加载`my-books`和`another-module`的依赖，然后执行模块正文代码，最后执行传入`YUI().use()`的回调函数。回调函数带回对象Y，Y对象里包含了模块对它执行的修改结果。
2. “异步模块定义”(AMD)
    异步模块定义（Asynchronous Module Definition）需要指定模块名称、依赖和工厂方法，执行时首先加载依赖，然后执行工厂方法。它们作为参数传入`define()`中。AMD与YUI的不同之处在于，每一个依赖都将作为独立的参数，传入到工厂方法中。  
    所谓的异步是指，依赖和模块本身加载的异步，也就是将模块同其依赖解耦合。也就是将各部分代码作为模块分开以提高可维护性。
    格式大概是这个样子的：
    {% codeblock lang:javascript %}
    define("module-name", [ "dependency1", "dependency2" ],
        function(dependency1, dependency2) {

        //function code

    });
    {% endcodeblock %}
    书上原文说道：
    > 每个被命名为的依赖最后都会创建一个对象，这个对象会被带入到工厂方法中。AMD以这种方式来尝试避免命名冲突，因为直接在模块中使用命名空间有可能发生命名冲突。和YUI模块中创建新的命名空间的方法不同，AMD模块则期望从工厂方法中返回它们的**公有接口**。如：
    {% codeblock lang:javascript %}
    define("my-books", [ "dependency1", "dependency2" ],
        function(dependency1, dependency2) {

        var Books = {};
        Books.MaintainableJavaScript = {
            author: "Nicholas"
        };

        return Books;
    });
    {% endcodeblock %}
    另外，AMD需要使用模块加载器来加载。比如RequireJS：
    {% codeblock lang:javascript %}
    require([ "my-books" ], function(books) {

        console.log(books.MaintainableJavaScript.author);

    });
    {% endcodeblock %}
    和AMD类似的JavaScript模块化规范还有CommonJS规范，上面引用的关于前端模块化的文章也有提到，CommonJS脱胎于Nodejs，比较适合服务器端或纯脚本编程，而在浏览器端不大适用。
