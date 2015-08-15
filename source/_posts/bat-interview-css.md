title: BAT及各大互联网公司2014前端笔试面试题：CSS篇
id: 40
categories:
  - 前端开发
date: 2014-11-21 00:00:00
tags:
---

1.有哪项方式可以对一个DOM设置它的CSS样式？
外部样式表，引入一个外部css文件
内部样式表，将css代码放在 &lt;head&gt; 标签内部
内联样式，将css样式直接定义在 HTML 元素内部
2.CSS都有哪些选择器？

派生选择器（用HTML标签申明）
id选择器（用DOM的ID申明）
类选择器（用一个样式类名申明）
属性选择器（用DOM的属性申明，属于CSS2，IE6不支持，不常用，不知道就算了）

除了前3种基本选择器，还有一些扩展选择器，包括

后代选择器（利用空格间隔，比如div .a{ }）
群组选择器（利用逗号间隔，比如p,div,#a{ }）

那么问题来了，CSS选择器的优先级是怎么样定义的？

基本原则：

一般而言，选择器越特殊，它的优先级越高。也就是选择器指向的越准确，它的优先级就越高。
复杂的计算方法：

用1表示派生选择器的优先级
用10表示类选择器的优先级
用100标示ID选择器的优先级
div.test1 .span var 优先级 1+10 +10 +1
span#xxx .songs li 优先级1+100 + 10 + 1
#xxx li 优先级 100 +1
3.CSS中可以通过哪些属性定义，使得一个DOM元素不显示在浏览器可视范围内？

最基本的：
设置display属性为none，或者设置visibility属性为hidden

技巧性：
设置宽高为0，设置透明度为0，设置z-index位置在-1000
4.超链接访问过后hover样式就不出现的问题是什么？如何解决？

答案：被点击访问过的超链接样式不在具有hover和active了,解决方法是改变CSS属性的排列顺序: L-V-H-A（link,visited,hover,active）
5.什么是Css Hack？ie6,7,8的hack分别是什么？

答案：针对不同的浏览器写不同的CSS code的过程，就是CSS hack。
7.行内元素和块级元素的具体区别是什么？行内元素的padding和margin可设置吗？

块级元素(block)特性：

总是独占一行，表现为另起一行开始，而且其后的元素也必须另起一行显示;
宽度(width)、高度(height)、内边距(padding)和外边距(margin)都可控制;
内联元素(inline)特性：
和相邻的内联元素在同一行;
宽度(width)、高度(height)、内边距的top/bottom(padding-top/padding-bottom)和外边距的top/bottom(margin-top/margin-bottom)都不可改变（也就是padding和margin的left和right是可以设置的），就是里面文字或图片的大小。
那么问题来了，浏览器还有默认的天生inline-block元素（拥有内在尺寸，可设置高宽，但不会自动换行），有哪些？

答案：&lt;input&gt; 、&lt;img&gt; 、&lt;button&gt; 、&lt;textarea&gt; 、&lt;label&gt;。
8.什么是外边距重叠？重叠的结果是什么？
答案：
外边距重叠就是margin-collapse。
在CSS当中，相邻的两个盒子（可能是兄弟关系也可能是祖先关系）的外边距可以结合成一个单独的外边距。这种合并外边距的方式被称为折叠，并且因而所结合成的外边距称为折叠外边距。

折叠结果遵循下列计算规则：

两个相邻的外边距都是正数时，折叠结果是它们两者之间较大的值。
两个相邻的外边距都是负数时，折叠结果是两者绝对值的较大值。
两个外边距一正一负时，折叠结果是两者的相加的和。

9.rgba()和opacity的透明效果有什么不同？
答案：
rgba()和opacity都能实现透明效果，但最大的不同是opacity作用于元素，以及元素内的所有内容的透明度，
而rgba()只作用于元素的颜色或其背景色。（设置rgba透明的元素的子元素不会继承透明效果！）

10.css中可以让文字在垂直和水平方向上重叠的两个属性是什么？
答案：
垂直方向：line-height
水平方向：letter-spacing

那么问题来了，关于letter-spacing的妙用知道有哪些么？
答案:可以用于消除inline-block元素间的换行符空格间隙问题。
11.如何垂直居中一个浮动元素？

```css
// 方法一：已知元素的高宽

#div1{
background-color:#6699FF;
width:200px;
height:200px;

position: absolute; //父元素需要相对定位
top: 50%;
left: 50%;
margin-top:-100px ; //二分之一的height，width
margin-left: -100px;
}

//方法二:未知元素的高宽

#div1{
width: 200px;
height: 200px;
background-color: #6699FF;

margin:auto;
position: absolute; //父元素需要相对定位
left: 0;
top: 0;
right: 0;
bottom: 0;
}
```