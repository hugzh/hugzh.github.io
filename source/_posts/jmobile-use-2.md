title: 移动前端框架jQuery Mobile 的介绍（二）
id: 51
categories:
  - javascript
  - 前端开发
date: 2015-01-29 00:00:00
tags:
---

上一篇主要是介绍了jMobile框架的基础知识，在这一篇里，我们主要是介绍
表单应用以及基于移动端的各种events。

一、表单基础：

在 jQuery Mobile 中，我们可以使用下列表单控件：
1、文本输入框
2、搜索输入框
3、单选按钮
4、复选框
5、选择菜单
6、滑动条
7、翻转拨动开关

1~5都是和PC前端的用法一样。其中，滑动条以及翻转拨动按钮是具有移动设备特色的表单。值得注意的是，由于
html5的使用，input标签的type值可以是mail、date、search，这是不同于PC端的使用，
非常方便。如需让标签和表单元素看起来更适应宽屏，可以用带有data-role="fieldcontain"
属性的
div或
fieldset元素包围 label/form 元素：

```html
<form method="post" action="demoform.html">
<div data-role="fieldcontain">
<label for="fname">姓:</label>
<input type="text" name="fname" id="fname">
<label for="lname">名:</label>
<input type="text" name="lname" id="lname">
</div>
</form>

```

&nbsp;

为了创建一个可以折叠的表单，可以将表单内容置于带有data-role="collapsible"

属性的fieldset中。
表单滑动条的使用如下：

```html

<input type="range" name="points" id="points" value="50" min="0" max="100"/>
```

&nbsp;

其中的value属性表示滑动条的默认值，min和max是滑动条的取值范围。如果想要高亮突出
显示滑动条的值，添加 data-highlight="true"。

至于拨动按钮开关，用法如下：

```html
<select name="switch" id="switch" data-role="slider">
<option value="on">On</option>
<option value="off">Off</option>
</select>

```

可以在option里面添加 "selected" 属性来设置选项中的一个为预选中状态（高亮突出显
示状态）。

二、页面事件：

在 jQuery Mobile 中, 使用pageinit 事件来设置代码脚本在DOM元素加载完成后开始
执行，所以要在任何新页面加载并创建是执行脚本，就需要绑定pageinit事件：

```javascript
<script>
$(document).on("pageinit","#pageone",function(){
});
</script>
```

点击事件：

```javascript
$("p").on("tap",function(){
$(this).hide();
});
```

长按事件：
```javascript

$("p").on("taphold",function(){
$(this).hide();
});
```

滑动事件：

```javascript
//简单滑动
$("p").on("swipe",function(){
$("span").text("Swipe detected!");
});

//向左滑动
$("p").on("swipeleft",function(){
alert("You swiped left!");
});

//向右滑动
$("p").on("swiperight",function(){
alert("You swiped right!");
});
```

滚动屏幕事件：

```javascript
//滚屏开始
$(document).on("scrollstart",function(){

});

//滚屏结束
$(document).on("scrollstop",function(){
});
```

另外，还有页面加载完成、失败、页面切换等事件，如果需要更加详细地了解jMobile的
用法，可以登录官方网站(http://jquerymobile.com/)去了解更多用法。