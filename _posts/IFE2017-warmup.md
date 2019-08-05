---
title:  'IFE2017 热身任务'
date: 2017-02-15 22:00
cover: '4.jpg'
category: "前端"
tags:
    - JavaScript
    - 前端
---
## 百度前端学院 IFE2017 热身任务

<!-- more -->

### 第零题
{% asset_img ife-unlock.png unlock%}
想想也就这种方法了，有点像智力题，没什么好说的。

### 第一题
{% asset_img ife-Step1.png %}
看到全屏白色，凭着很久以前逛贴吧论坛的经验，下意识地按了一下Ctrl+A。不过这很不前端，F12才是正确答案。  
这一条内容无非也就是加密解密，base64解密之后会得到第二题URL的后缀。复制粘贴过关。

### 第二题
{% asset_img ife-Step2.png %}
这道题有两个用单引号包裹起来的关键字：窗`window`和高度`innerHeight`。控制台里取出这个属性的值，然后把密码盘调成这个值就行了。可以先调整一下窗口大小，比如520px，这样只需要改一个值就行了，省时省力。

### 第三题
{% asset_img ife-Step3.png %}
这个应该完全是CSS3的内容。虽然第一个字母并不需要CSS3就能做好。
{% codeblock lang:css %}
.letter-i {
    -webkit-transform: translate(561px, -191px);
}
.letter-f {
    -webkit-transform: translate(571px, -99px);
}
.letter-e {
    -webkit-transform: rotate3d(0.181, 1, 0, 180deg) translate(-592px, 17px);
}
{% endcodeblock %}
其中第三个字母E有点意思。可以直接使用`rotateX`和`rotateY`来实现翻转和旋转。这里我直接将两个操作使用一个方法来实现，即使用`rotate3d(x, y, z, deg)`。这里的`x`、`y`、`z`可以理解成，旋转`deg`度这一操作分别在关于x轴、关于y轴、关于z轴这三个方向上的效果系数。系数越接近1，实际作用就越趋近180度。现在只想关于y轴翻转180°，则令y=1，z=0，deg=180，固定这三个参数后，调整x的效果系数，使其旋转到合适角度。最后使用`translate`调整位置。
{% asset_img ife-Step3-answer.png %}

### 第四题
{% asset_img ife-Step4.png %}
这一题最为麻烦也最为耗时。有两个tips：
1. 在图上点击可以在控制台得到点坐标，方便寻找路径
2. 尽量控制小球靠边走，如果需要折回去吃新刷新的星星的话，两次转弯之间至少要隔5px，否则会因为延迟不能转弯撞墙。
