---
title: "定制化Popover的实现思路"
date: 2017-08-08 0800
cover: "2.jpg"
category: "前端"
tags:
    - React
    - CSS
---
# 定制化Popover的实现思路
记得春招的时候，有同学分享过一个阿里的面试题：要在半个小时内实现一个Modal框还是弹出框来着。当时同学的看法是，如果事先没有做过类似的工作，从头开始想是很难在半个小时内完成的。

现在项目中遇到了这个问题，把实现的过程记录下来。这个组件从代码量上看的确是不难，但是如果没有经验而自己去想，思考的过程还是很费时间的。

## 动手之前的思考
当时真的是花了比较长的时间想，从要做这个组件，到完成用了快两天。也是没有经验，想得太久。

首先，我遇到的场景是，需要做一个弹出框，这个弹出框的接口需要有：
1. 位置：弹出框的位置是经过其他组件计算得出的，它并不像AntD那样由某个固定按钮触发弹出，它的确切位置也不能事先知道，全部取决于用户的操作。
2. 内容：弹出框的内容多变，少的情况只有两个svg icon按钮，多的时候可能有若干个选项卡tab这样子，也是不确定的
3. 显示控制：显隐需要由父容器组件控制
4. 挂载点：有些情况下，一个页面内需要显示Popover的地方不止一个，比如一个三栏布局，要求三个栏内可能同时显示三个Popover，这样就需要分别设定三个Popover组件各自的挂载点，使得它们不会重合在某一个容器内。

第4点是在开始考虑的时候，参考了AntD，在看它里面Popover源码的时候发现，传入该组件的`props`里有一个获取挂载DOM元素的回调函数。开始并没有这个选项有什么作用，后来猜测有以下几个场景可能需要设置这个值：
1. 触发显示Popover的按钮或其他元素可能会在这个Popover之后被渲染，那么这个Popover至少应该有一个预先存在的挂载点。默认的是`body`，这一点可以理解。
2. 就像上面需求4所描述的，一个页面存在多个Popover的情况
3. Popover弹出之后，需要根据页面中存在的某个元素确定位置，可以指定Popover就挂载在这个元素上

除了这些数据接口，还需要的考虑的是交互样式：
1. 它应该位于某个确定的容器内
2. 具有气泡形状，最简单的就是圆角矩形加个小三角
3. 显示/消失过渡效果

## Do it！
想好这些要点之后，实现的时候还有一些坑。

首先，写这个组件的显示逻辑和JSX没什么难度：
```javascript
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import classNames from 'classnames';
import style from './style.css';

class Popover extends Component {
  static defaultProps = {
    position: {
      left: 0,
      top: 0,
    },
    visible: 'false',
  };

  clickInterceptor = ev => {
    ev.stopPropagation();
    ev.preventDefault();
  };

  render() {
    const { content, position, visible } = this.props;
    const cls = classNames({
      popover: true,
      'popover-fadein': visible,
      'popover-hidden': !visible,
    });
    return (
      <div
        className={cls}
        style={{
          left: position.left,
          top: position.top,
        }}
        onMouseUp={ev => this.clickInterceptor(ev)}
        onMouseDown={ev => this.clickInterceptor(ev)}
        onClick={ev => this.clickInterceptor(ev)}
      >
        {content}
        <div
          className={classNames({
            'popover-triangle': true,
            'popover-triangle-back': true,
          })}
        />
        <div
          className={classNames({
            'popover-triangle': true,
            'popover-triangle-front': true,
          })}
        />
      </div>
    );
  }
}

Popover.propTypes = {
  content: PropTypes.instanceOf(Object).isRequired,
  position: PropTypes.instanceOf(Object),
  visible: PropTypes.bool,
};

export default Popover;
```
其中类方法里有个点击拦截器(`clickInterceptor`)，这个东西是项目需求，不让点击在Popover里的操作扩散出去影响用户之前操作的结果。

有点点坑的地方是样式。这个实例里面主要涉及到三个有意思的点：
1. CSS画三角，简单有趣
2. 不同尺寸的内容，使得Popover形变时，要控制变形锚点
3. 显隐、位移时的过渡动画

这三个点还是很有趣的，我一个一个记录。首先是`popover`本身：
```css
.popover {
  position: absolute;
  padding: 0.5em;
  border-radius: 8px;
  border: 1px solid #ccc;
  background: #fff;
  max-width: 300px;
  opacity: 0;
  transform-origin: 50% 100%;
  transform: translate(-50%, -100%);
  box-shadow: 0 1px 6px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}
```
首先我这里实现的弹出框组件，只用在一个固定的父容器组件中，所以这里没有设计挂载点接口，同时它的位置也是`absolute`。  
默认情况下它的透明度是0，也就是隐藏在页面上的。这主要是为了淡入过渡效果而设定的。  
```css
  transform-origin: 50% 100%;
  transform: translate(-50%, -100%);
```
这两行就是为了满足上面说的第二点而设置的。`transform-origin`的作用是：
> transform-origin CSS属性让你更改一个元素变形的原点。

需要注意的是它作为Working Draft阶段的CSS特性，还没有一个通用的使用方式。  

Chrome | Firefox | IE | Opera | Safari
-------|---------|----|-------|-------
`-webkit`|`-moz`|`-ms`|`-o`|`-webkit`

其中三值语法Opera还没有实现。参考[MDN `transform-origin`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-origin)

设置了这两个属性之后，当Popover里的内容变高变宽的时候，它的形变中心将是下边缘中点，形变方向则是左右平均延伸，向上延伸。如果不设置，则会得到一个锚点为左上角顶点，向右向下延伸的弹出框。当然，这可能根据需求变化而变化。

有了这个基础元素之后，我们要为它添加气泡的小尖角：
```css
.popover-triangle {
  position: absolute;
  left: 50%;
  width: 0;
  height: 0;
}

.popover-triangle-front {
  border-top: 8px solid #fff;
  border-left: 8px solid transparent;
  border-right: 8px solid transparent;
  margin-left: -8px;
  bottom: -8px;
}

.popover-triangle-back {
  border-top: 10px solid #ccc;
  border-left: 10px solid transparent;
  border-right: 10px solid transparent;
  margin-left: -10px;
  bottom: -10px;
}
```
这三个元素样式原理就是使用`border`相关的属性画两个三角叠加起来，用父容器控制位置，其中一个三角是前景白色尖角，另一个稍大，作为阴影放在后面。

这样一个Popover组件大致就完成了。但是还需要一些过渡效果。正如上面所说，Popover的位置是经常变化的，我不想让它每次都闪现，所以需要这样：
```css
.popover-fadein {
  display: block;
  opacity: 1;
  transition: left 200ms ease, top 200ms ease, opacity 300ms ease-in-out;
}

.popover-hidden {
  opacity: 0;
  max-width: 0;
  padding: 0;
  transition: max-width 100ms ease 300ms, border-radius 100ms ease 300ms, padding 100ms ease 300s, opacity 200ms ease-in-out;
}
```
添加这两个类的目的在于：
1. 赋予Popover淡入淡出的效果
2. 在Popover元素移动时，按照`ease`定义的贝塞尔函数设定其移动速度

记录一下`transition`的语法
```css
transition: <property> <duration> <timing-function> <delay>;
```

到这里，一个基本满足需求的Popover就做完啦！还是收获颇丰的。

关于这个组件，还有一些问题。可以看出Popover的最大宽度为300px，问题就在于当Popover出现在所在容器的最左端和最右端时，它可能显示不完整。这是需要改进的一个问题。思路就是当尖角左侧或右侧的宽度不足以显示Popover一半宽度内容的时候，将尖角和位移、形变中心移到Popover左下角，形成一个变形时向右上延伸的Popover。

由于时间有限，没有添加这个特性。不过思路更加重要一点！:smile:
