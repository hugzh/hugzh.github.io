title: Flexbox 布局
date: 2015-09-02 11:57:07
tags: 
    - css
categories:
    - 前端开发
---

# 1、简述

Flex布局也叫“弹性盒子布局”，用来为盒状模型提供最大的灵活性。弹性布局的主要思想是让容器有能力来改变项目的宽度和高度，以填满可用空间（主要是为了容纳所有类型的显示设备和屏幕尺寸）的能力。
最重要的是弹性盒子布局与方向无关，相对于常规的布局（块是垂直和内联水平为基础），很显然，这些工作以及网页设计缺乏灵活性，无法支持大型和复杂的应用程序（特别当它涉及到改变方向，缩放、拉伸和收缩等）。

由于flexbox是一个整体模块，而不是单一的一个属性，它涉及到了很多东西，包括它的整个属性集。它们之中有一些是在父容器上设置，而有一些则是在子容器上设置。
<!--more-->
## 2、flexbox属性详解
首先，需要看看整个弹性盒子的组成，先看下面一个图：
![flexbox](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071004.png)

采用Flex布局的元素，称为Flex容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为Flex项目（flex item），简称"项目"。
容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。

## 3、容器的flex属性

首先，定义一个flex容器的方法:

```css
.container {
  display: -webkit-flex;
  display: flex; /* or inline-flex */
}
```
容器的属性包括6个：
* flex-direction
* flex-wrap
* flex-flow
* justify-content
* align-items
* align-content
### 3.1、flex-direction属性
该属性决定主轴的方向，包括4个值：
* row（默认值）：主轴为水平方向，起点在左端。
* row-reverse：主轴为水平方向，起点在右端。
* column：主轴为垂直方向，起点在上沿。
* column-reverse：主轴为垂直方向，起点在下沿。
![flex-direction](https://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-direction1.svg)

```css
.container {
  flex-direction: row | row-reverse | column | column-reverse;
}
```
### 3.2、flex-wrap属性
默认情况下，项目都排在一条线（又称"轴线"）上。flex-wrap属性定义，如果一条轴线排不下，如何换行。
一共3个值，
* nowrap:(默认)不换行
* wrap：换行
* wrap-reverse：按照从左往右，换行之后从右到左，再换行从左到右这样的顺序排列
![flex-wrap](https://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-wrap.svg)
```css
.container{
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

### 3.3、flex-flow属性
flex-flow属性是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap。

```css
.container{
    flex-flow: <‘flex-direction’> || <‘flex-wrap’>
}
```

### 3.4、justify-content属性
定义主轴上面的对齐方式，帮助灵活或者不灵活的元素在未占满整行的时候分布多余的空间。包括5个值：
* flex-start（默认值）：主轴开始方向对齐
* flex-end：主轴结束方向对齐
* center： 居中
* space-between：两端对齐，项目之间的间隔都相等。
* space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

![justify-content](https://cdn.css-tricks.com/wp-content/uploads/2014/05/align-items.svg)
```css
.container {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

### 3.5、align-items属性
align-items属性定义项目在交叉轴上如何对齐。包括5个值：
* flex-start：交叉轴的起点对齐。
* flex-end：交叉轴的终点对齐。
* center：交叉轴的中点对齐。
* baseline: 项目的第一行文字的基线对齐。
* stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

![align-items](https://cdn.css-tricks.com/wp-content/uploads/2014/05/align-items.svg)

```css
.container {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
```

### 3.6、align-content属性
定义交叉抽上面有多余空间时的对齐方式。类似于主轴上面对齐的justify-content属性。如果项目只有一根轴线，该属性不起作用。
该属性可以取6个值：
* flex-start：与交叉轴的起点对齐。
* flex-end：与交叉轴的终点对齐。
* center：与交叉轴的中点对齐。
* space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
* space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
* stretch（默认值）：轴线占满整个交叉轴。

![align-content](https://cdn.css-tricks.com/wp-content/uploads/2013/04/align-content.svg)

```css
.container {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

## 4、子元素（项目）的属性
弹性布局元素的子元素的属性包括6个：
* order
* flex-grow
* flex-shrink
* flex-basis
* flex
* align-self


### 4.1、order属性
子元素默认是按照代码声明的顺序铺层的，可以通过order值指定某个子元素的排序。order的取值为整数（可以是负数，默认0）
![order](https://cdn.css-tricks.com/wp-content/uploads/2013/04/order-2.svg)

```css
.item {
  order: <integer>;
}
```

### 4.2、flex-grow属性
该属性定义弹性布局子元素的“生长”能力。

如果所有项目的flex-grow属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的flex-grow属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。
![flex-grow](https://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-grow.svg)

```css
.item {
  flex-grow: <number>; /* default 0 */
}
```

### 4.3、flex-shrink属性
flex-shrink属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。不允许为负数。

如果所有项目的flex-shrink属性都为1，当空间不足时，都将等比例缩小。如果一个项目的flex-shrink属性为0，其他项目都为1，则空间不足时，前者不缩小。

```css
.item {
  flex-shrink: <number>; /* default 1 */
}
```

### 4.4、flex-basis属性
定义剩余空间未分配之前的元素默认尺寸。
```css
.item {
  flex-basis: <length> | auto; /* default auto */
}
```

### 4.5、flex属性
flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选。
建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会自动推算其它相关值。
```css
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```

### 4.6、align-self属性
align-self属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。该元素包括6个值。
![align-self](https://cdn.css-tricks.com/wp-content/uploads/2014/05/align-self.svg)

```css
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

本文参考CSS-TRICKS上的文章[A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)、还有阮一峰老师的文章[Flex 布局教程](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html?utm_source=tuicool)



