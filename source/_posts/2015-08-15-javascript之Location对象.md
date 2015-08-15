title: javascript之Location对象
date: 2015-08-15 12:55:22
tags:
    - hash
    - location
categories:
    - javascript
    - 前端开发
---

#  Location对象

location对象是最有用的BOM对象之一，它提供了与当前窗口中加载的文档有关的信息，还提供了一些导航功能。location对象既是window对象的属性，也是document对象的属性，window.location和document.location是引用同一个对象。

## location对象的所有属性
* hash          //返回URL中的hash(#后面跟的零个或者多个字符),如果URL中不包含散列，则返回空字符串
* host          //返回服务器的名称和端口（如果有）
* hostname      //返回不带端口的服务器名称
* href          //返回当前加载页面的完整URL，location对象的toString()方法也是返回这个值
* pathname      //返回URL中的路径
* port          //返回URL中指定的端口号，没有包含则为空
* ptotocol      //返回这个页面使用的协议
* search        //返回查询字符串，比如“?q=javascript”
我想在这里讨论的是location对象的hash属性及其用法

## 关于hash
#### “#”的含义
“#”代表网页中的一个位置。其右面的字符，就是该位置的标识符。比如,"http://www.example.com/index.html#print"
就代表网页index.html的print位置。浏览器读取这个URL后，会自动将print位置滚动至可视区域。
为网页位置指定标识符，有两个方法。一是使用锚点，比如
```html 
<a name="print"></a>
```
二是使用id属性，比如
```html
<div id="print" ></div>
```

#### HTTP请求不包括"#"
“#”是用来指导浏览器动作的，对服务器端完全无用。所以，HTTP请求中不包括“#”。在第一个“#”后面出现的任何字符，都会被浏览器解读为位置标识符。这意味着，这些字符都不会被发送到服务器端。

#### 改变“#”后面的字符不触发网页重载
单单改变“#”后的部分，浏览器只会滚动到相应位置，不会重新加载网

#### 改变“#”会改变浏览器的访问历史
每一次改变“#”后的部分，都会在浏览器的访问历史中增加一个记录，使用"后退"按钮，就可以回到上一个位置。
这对于ajax应用程序特别有用，可以用不同的“#”值，表示不同的访问状态，然后向用户给出可以访问某个状态的链接。
值得注意的是，上述规则对IE 6和IE 7不成立，它们不会因为#的改变而增加历史记录。

#### 监听hash值变化的方法
```javascript
window.onhashchange = func;
<body onhashchange="func();">
window.addEventListener("hashchange", func, false);
```
其中，onhashchange是一个html5的事件。当“#”值发生变化时，就会触发这个事件。IE8+、Firefox 3.6+、Chrome 5+、Safari 4.0+支持该事件。对于不支持onhashchange的浏览器，可以用setInterval监控location.hash的变化。

#### Google抓取“#”的机制
默认情况下，Google的网络蜘蛛忽视URL的#部分。
但是，Google还规定，如果你希望Ajax生成的内容被浏览引擎读取，那么URL中可以使用"#!"，Google会自动将其后面的内容转成查询字符串_escaped_fragment_的值。