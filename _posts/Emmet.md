---
title: Emmet快捷键总结
date: 2016-04-13 10:00:00
cover: '5.jpg'
category: "Editor"
tags:
    - Emmet
---

## Emmet快捷键总结
### HTML标签
#### 生成基本文档结构
`html:5` 或者 ! 生成 HTML5 结构
`html:xt` 生成 HTML4 过渡型
`html:4s` 生成 HTML4 严格型
<!--more-->
#### div with id, class
`span.ddd`
`ul#ccc.ddd`

#### 父代、后代与兄弟标签
父代标签：
`div>ul>li^span`
后代有序标签：
`div.aaa>ul>li`
后代无序标签：
`div.abc>ol.li`
兄弟标签：
`di+p+bq`

#### 副本与分组
`ul>li*5`
`div>(header>ul>li*2>a)+footer>p`

#### 自定义属性标签
`a[href="http://github.com" title="自定义链接属性"]`

#### 内容编号占位符$
生成带有编号的单位，也可以使用多位编号：
`ul>li.item$*5`
`ul>li.item&&&*5`
也可以使用倒序：
`ul>li.item$@-*5`
`ul>li.item$@3*5`

#### 提供标签的Value/Text
`a[href="github.com"]{标签的Value}`

### CSS的key-value
`position: absolute;`---> `posa`
`width: 100px;`---> `w100`
`width: 100%`--->`w100p`
`width: 100em`--->`w100e`

`margin: 10px 10px`--->`m10-10`
