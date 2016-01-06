title: HTML5里的placeholder属性
tags:
  - css
  - html5
categories:
  - 前端开发
date: 2015-03-26 23:33:49
---

HTML5里新引入很多有趣的新特征；有些体现在HTML里，有些是JavaScript API，全部非常的有用。其中我最喜欢的一个特征就是文本框(INPUT)里的 placeholder 属性。placeholder属性能够让你在文本框里显示提示信息，一旦你在文本框里输入了什么信息，提示信息就会隐藏。你以前可能无数次看到这种效果，但那些大部分是用JavaScript里实现的，而现在，HTML5提供了原生支持，而且效果更好！
<!--more-->
## 用法:

 需要做的只是在文本框的声明标签上加入 placeholder 属性。完全不需要JavaScript来创造这种效果。

```
<input type="text" name="first_name" placeholder="你的姓名..." />
```

## 检查浏览器是否支持Placeholder属性：

因为 placeholder 是一种新属性，很有必要检查一下你的浏览器是否支持它，比如IE6、IE8肯定是不支持的：

```
function hasPlaceholderSupport() {
	var input = document.createElement('input');
	return ('placeholder' in input);
}
```
如果用户的浏览器不支持 placeholder 特征，你需要借助MooTools, Dojo, 或其它JavaScript工具来实现它

```
/* mootools ftw! */
var firstNameBox = $('first_name'),
	message = firstNameBox.get('placeholder');
firstNameBox.addEvents({
	focus: function() {
		if(firstNameBox.value == message) { searchBox.value = ''; }
	},
	blur: function() {
		if(firstNameBox.value == '') { searchBox.value = message; }
	}
});

```

## 可以对样式进行CSS美化：
```
::-webkit-input-placeholder { color:#f00; }
::-moz-placeholder { color:#f00; } /* firefox 19+ */
:-ms-input-placeholder { color:#f00; } /* ie */
input:-moz-placeholder { color:#f00; }

/* individual: webkit */
#field2::-webkit-input-placeholder { color:#00f; }
#field3::-webkit-input-placeholder { color:#090; background:lightgreen; text-transform:uppercase; }
#field4::-webkit-input-placeholder { font-style:italic; text-decoration:overline; letter-spacing:3px; color:#999; }

/* individual: mozilla */
#field2::-moz-placeholder { color:#00f; }
#field3::-moz-placeholder { color:#090; background:lightgreen; text-transform:uppercase; }
#field4::-moz-placeholder { font-style:italic; text-decoration:overline; letter-spacing:3px; color:#999; }
```

##
你可以控制placeholder文字的字体、颜色和风格。你甚至可以将文本框的placeholder以动画方式显示。 美化你的文本框都是些看起来很小的事情，但对于一些交互式的网站来说，成功的关键就在于细节。现在IE10里也只支持placeholder了，相信越来越多的人会使用这种原生的placeholder效果。

(本文转载自 http://www.webhek.com/html5-placeholder?rel=http://geek.csdn.net)
