title: 前端笔试面试中的常用到的CSS知识点总结
date: 2015-11-16 22:41:23
tags:
    - css
categories:
    - 前端开发
---
*在前端开发领域，CSS是比较特殊的一种存在。理论上说，它是一种属性大杂烩，并不属于编程语言的范畴，当然SASS和LESS这两种预编译机制除外。所以CSS的学习是需要不断地去记忆和积累的。在前端笔试和面试过程中，CSS的考察是比较琐碎的，而且往往考察点比较偏，需要我们平时养成做笔记的习惯。这篇文章是我之前发布在慕课社区的一篇小笔记，原文[前端笔试面试中的常用知识点总结(CSS)](http://www.imooc.com/article/2181)，主要记录的是自己在校招过程当中关于CSS知识的总结。* 
<!--more-->
### 1、CSS选择器的优先级
!important  > 内联 > id选择器 > 类选择器 > 标签选择器
多个类选择器叠加（256）之后的优先级大于一个id选择器
!important 用于强调CSS属性具有最高的优先级。IE6不支持这种用法。
CSS选择器的种类：
1.id选择器(# myid)
2.类选择器(.myclassname)
3.标签选择器(div, h1, p)
4.相邻选择器(h1 + p)
5.子选择器(ul > li)
6.后代选择器(li a)
7.通配符选择器( * )
8.属性选择器(a[rel = "external"])
9.伪类选择器(a: hover, li:nth-child)
### 2、CSS优先级权重计算法
CSS优先级的计算规则如下：
元素标签中定义的样式（Style属性）,加1,0,0,0
每个ID选择符(如 #id),加0,1,0,0
每个Class选择符、每个属性选择符、每个伪类（:hover）加0,0,1,0
每个元素选择符（如p）或伪元素选择符(如 :firstchild)等，加0,0,0,1
然后，将这四个数字分别累加，就得到每个CSS定义的优先级的值
然后从左到右逐位比较大小，数字大的CSS样式的优先级就高。
### 3、超链接访问过后hover样式就不出现的问题是什么？如何解决？
被点击访问过的超链接样式不在具有hover和active了,解决方法是改变CSS属性的排列顺序: L-V-H-A（link,visited,hover,active）
### 4、什么是Css Hack？ie6,7,8的hack分别是什么？
针对不同的浏览器写不同的CSS code的过程，就是CSS hack。常见的hack手法：
```
#test       {   
        width:300px;   
        height:300px;   
        background-color:blue;      /*firefox*/
        background-color:red\9;      /*all ie*/
        background-color:yellow\0;    /*ie8*/
        +background-color:pink;        /*ie7*/
        _background-color:orange;       /*ie6*/    }  
        :root #test { background-color:purple\9; }  /*ie9*/
    @media all and (min-width:0px){ 
          #test {background-color:black\0;} 
     }  /*opera*/
    @media screen and (-webkit-min-device-pixel-ratio:0){ 
#test {background-color:gray;} 
}       /*chrome and safari*/
```
### 5、描述一个"reset"的CSS文件并如何使用它。知道normalize.css吗？你了解他们的不同之处？
不同的浏览器对一些元素有不同的默认样式，如果你不处理，在不同的浏览器下会存在必要的风险，或者更有戏剧性的性发生。
　　你可能会用Normalize来代替你的重置样式文件。它没有重置所有的样式风格，但仅提供了一套合理的默认样式值。既能让众多浏览器达到一致和合理，但又不扰乱其他的东西（如粗体的标题）。
### 6、display:none与visibility:hidden的区别是什么？
display : 隐藏对应的元素但不挤占该元素原来的空间。
visibility: 隐藏对应的元素并且挤占该元素原来的空间。
 　　即是，使用CSS display:none属性后，HTML元素（对象）的宽度、高度等各种属性值都将“丢失”;而使用visibility:hidden属性后，HTML元素（对象）仅仅是在视觉上看不见（完全透明），而它所占据的空间位置仍然存在。
### 7、CSS盒子模型
Box Model规定了元素框处理元素内容（element content）、内边距（padding）、边框（border） 和 外边距（margin） 的方式。盒子模型有 block,inline-block,inline,flex,table-cell等几种显示方式。
![CSS盒模型](http://img.mukewang.com/5649f2ab0001b95904640464.png)
如图所示，元素的高度和宽度指的是盒子模型的content区域的高度和宽度，不包括内边距以外的部分。(IE的content则是包含border、内边距和content)。盒子模型有两种，W3C标准盒子模型（content-box）以及IE盒子模型（border-box）。
### 8、相对定位relative、浮动float以及绝对定位absolute的区别？
1）相对定位是在常规流中的定位。一旦一个框按照常规流或者是浮动得到定位，它还可以相对该位置而偏移， 这就是相对定位。偏移后，在常规流中依然占据原有位置，偏移量并不会对后续的block产生挤占，后续的块就好像没有感知到偏移，依然按照常规流位置排位。
2）绝对定位（Absolute positioning）元素定位的参照物是其包含块（第一个relative定位的祖先元素）进行定位，不一定是其父元素。绝对定位元素完全脱离了常规流，每个绝对定位的框都可以看做一个单独的图层，不会对其他层框的定位产生影响。绝对元素定位的 top 和 left 值跟绝对元素未脱离常规流之前在常规流中位置有关。
3）浮动框就是一个框在当前行被向左或向右挪动(偏移)，它不在常规流中。在该浮动框之前或之后创建的非定位框垂直排列，就好象浮动框并不存在一样。当前行里浮动框前的任何内容，都将被重新排列到该浮动另一侧的第一个可用行里。
### 9、清除浮动的用法？
在非IE浏览器（如Firefox）下，当容器的高度为auto，且容器的内容中有浮动（float为left或right）的元素，在这种情况下，容器的高度不能自动伸长以适应内容的高度，使得内容溢出到容器外面而影响（甚至破坏）布局的现象。这个现象叫浮动溢出，为了防止这个现象的出现而进行的CSS处理，就叫CSS清除浮动。
使用方法：
1、在浮动元素后使用一个空元素如<div class="clear"></div>，并在CSS中赋予.clear{clear:both;}属性即可清理浮动。
2、给浮动元素的容器添加overflow:hidden;或overflow:auto;可以清除浮动
3、什么都不做，给浮动元素后面的元素添加clear属性。
4、利用伪类清除浮动，给浮动元素的容器添加一个clearfix的class，然 后给这个class添加一个:after伪元素实现元素末尾添加一个看不见的块元素（Block element）清理浮动。
```
//一种常见利用伪类清除浮动的代码
 .clearfix:after {
    content:".";       //这里利用到了content属性
    display:block; 
    height:0;
    visibility:hidden; 
    clear:both; 
}
.clearfix { 
    *zoom:1; 
}
```
### 10、BOX模型的margin在IE比在其他浏览器大2px的解决办法

div{margin:30px!important;margin:28px;}
注意这两个margin的顺序一定不能写反，据阿捷的说法!important这个 属性IE不能识别，但别的浏览器可以识别。所以在IE下其实解释成这样：
```
div{maring:30px;margin:28px}
```
重复定义的话按照最后一个来执行，所以不可以只写
```
margin:XXpx!important;
```
也可以这样子hack
```
div { _margin:28px;margin:28px\9;margin:30px;}
```
### 11、IE 3像素Bug的产生原因及解决办法
当在同一行相邻出现一个浮动的块元素和一个非浮动的块元素时，两块元素之间就会出现3px的裂痕。
解决办法就是：对其中一个元素添加“_margin-*:-3px”的CSS属性，或者将另外一个非浮动的元素设为浮动。

### 12、IE6 双边距Bug的产生原因及解决办法
当我们在浮动元素上面添加margin-*的属性的时候，比如在float:left的元素CSS添加margin-left:50px，那么最终在IE6下面的显示就是该元素的margin-left最终变为100px，边距翻倍。
解决办法：添加_display:inline

### 13、FOUC - Flash Of Unstyled Content 文档样式闪烁
```
<style type="text/css" media="all">@import "../fouc.css";</style> 
```
而引用CSS文件的@import就是造成这个问题的罪魁祸首。IE会先加载整个HTML文档的DOM，然后再去导入外部的CSS文件，因此，在页面DOM加载完成到CSS导入完成中间会有一段时间页面上的内容是没有样式的，这段时间的长短跟网速，电脑速度都有关系。
解决方法简单的出奇，只要在 head 之间加入一个 link 或者 script 元素就可以了。

### 14、文本溢出省略的处理方法
1)单行文本溢出
```
.inline{
  overflow:hidden;
         white-space:nowrap;
         text-overflow:ellipsis; 
}
```
2)多行文本溢出
```
.foo{
  display:-webkit-box!important;
  overflow:hidden;
  text-overflow:ellipsis;
  work-break:break-all;
  -webkit-box-orient:vertical; /*方向*/
  -webkit-line-clamp:4; /*显示多少行文本*/
}
```
但是这里只适用于webkit内核浏览器，不兼容IE和其他非webkit浏览器。

### 15、CSS实现垂直水平居中
方法1：
```
.parent {
          width:800px;
          height:500px;
          border:2px solid #000;
          position:relative;
}
 .child {
        width:200px;
        height:200px;
        margin: auto;  
        position: absolute;  
        top: 0; left: 0; bottom: 0; right: 0; 
        background-color: red;
}
```
方法2：
```
    .parent {
        width:800px;
        height:500px;
        border:2px solid #000;
        display:table-cell;
        vertical-align:middle;
        text-align: center;
    }
    .child {
        width:200px;
        height:200px;
        display:inline-block;
        background-color: red;
    }
```
方法3：
```
    .parent {
        width:800px;
        height:500px;
        border:2px solid #000;
        display:flex;
        justify-content:center;
        align-items:center;
    }
    .child {
        width:200px;
        height:200px;
        background-color: red;
    }
```
方法4：
```
    .parent {
            width:800px;
            height:500px;
            border:2px solid #000;
            position:relative;
        }
    .child {
        width:300px;
        height:200px;
        margin:auto;
        position:absolute;
    //设定水平和垂直偏移父元素的50%，
    //再根据实际长度将子元素上左挪回一半大小
        left:50%;
        top:50%;
        margin-left: -150px;
        margin-top:-100px;
        background-color: red;
    }  
```
### 16、BFC
块格式上下文是页面CSS 视觉渲染的一部分，用于决定块盒子的布局及浮动相互影响范围的一个区域。

BFC的创建方法：
根元素或其它包含它的元素；
浮动 (元素的float不为none)；
绝对定位元素 (元素的position为absolute或fixed)；
行内块inline-blocks(元素的 display: inline-block)；
表格单元格(元素的display: table-cell，HTML表格单元格默认属性)；
overflow的值不为visible的元素；
弹性盒 flex boxes (元素的display: flex或inline-flex)；

### 17、响应式图片的CSS
所谓响应式图片，就是根据图片所在容器的大小来实时地按比例缩放。实现的办法非常简单，只需要设置img标签的width:100%即可。
```
img {
width:100%;
}
```
如果 max-width 属性设置为 100%, 图片永远不会大于其原始大小：
```
img {
max-width: 100%;
}
```
### 18、CSS响应式网格系统原理
总的来说，网格系统就是利用百分比把视口等分为12个，然后利用float属性使之并列显示。
```
* {
    box-sizing: border-box;
}
[class*="col-"] {
    float: left;
    padding: 15px;
    border: 1px solid red;
}
.col-1 {width: 8.33%;}
.col-2 {width: 16.66%;}
.col-3 {width: 25%;}
.col-4 {width: 33.33%;}
.col-5 {width: 41.66%;}
.col-6 {width: 50%;}
.col-7 {width: 58.33%;}
.col-8 {width: 66.66%;}
.col-9 {width: 75%;}
.col-10 {width: 83.33%;}
.col-11 {width: 91.66%;}
.col-12 {width: 100%;}
```
### 19、用LESS循环实现栅格类
```
.generate-columns(12);
.generate-columns(@n; @i: 1) when (@i =< @n) {
  .column-@{i} {
    width: (@i * 100% / @n);
  }
.generate-columns(@n, (@i + 1));
}
```
### 20、使用perspective属性和transform：perspective()函数的区别
以上的两种书写格式都会生成一个3D立体效果，但是它们之间有一些区别。使用功能函数的方式可以很方便的在单个元素上应用3D transform，但是当在多个元素上使用时，转换的元素不会得到预期的效果。如果你为不同的元素使用相同的transform，每一个元素都会有自己的消失点。为了解决这个问题，可以在父元素上使用perspective属性，这样，所有的子元素都可以共享同一个3D空间。
perspective-origin: 25% 75%，等价于perspective-origin x和perspective-origin y的合体，用来设置镜头聚焦点的位置(舞台中心)
transform-style: preserve-3d,transform-style属性也是3D效果中经常使用的，其两个参数，flat|preserve-3d. 前者flat为默认值，表示平面的；后者preserve-3d表示3D透视。
backface-visibility：hidden|visibility设置背后元素是否可见
