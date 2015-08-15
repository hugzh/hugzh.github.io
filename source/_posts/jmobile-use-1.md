title: 移动前端框架jQuery Mobile 的介绍（一）
id: 48
categories:
  - javascript
  - 前端开发
date: 2015-01-28 00:00:00
tags:
---

简介：
jQuery Mobile 是用于创建移动 Web应用的前端开发框架，可以应用于智能手机与平板电脑。是一个比较好用的前端框架。
jMobile适用于ios，android，black berry，windows phone等主流移动终端操作系统。

jQuery Mobile 的安装：
安装非常简单，就像jQuery的安装一样，可以直接把文件下载到本地，然后再html文件的head标签内引用，也可以直接通过
CDN 中加载。需要包含三个文件jquery.mobile.min.css、jquery.js以及jquery.mobile.min.js。由于jMobile是自带一套UI的框架，
所以第一个css文件是必须要包含进来的。

一个简单的jMobile页面包括三个部分——header、content和footer。通过data-role来实现，比如:

```html

<div data-role="header">
<h1>头</h1>
</div>

```

```html

<div data-role="content">
<p>主体</p>
</div>

```

```html

<div data-role="footer">
<h1>底部</h1>
</div>

```

其中，jQuery Mobile 依赖 HTML5 data-* 属性来支持各种 UI 元素、过渡和页面结构：
data-role="page" 是在浏览器中显示的页面。
data-role="header" 是在页面顶部创建的工具条 (通常用于标题或者搜索按钮)
data-role="content" 定义了页面的内容，比如文本， 图片，表单，按钮等。
data-role="footer" 用于创建页面底部工具条。
在这些容器中你可以添加任何 HTML 元素 - 段落, 图片, 标题, 列表等。

在页面的切换方式使用data-transition来定义，可以包括从上到下滑动，从左到右滑动，抛出页面，弹窗等多种形式。
在导航元素上，data-role="navbar"加图标data-icon可以实现导航栏。每一个的导航元素可以通过class="ui-btn-active"
来实现按钮激活状态。在布局方面，可以使用网格和可折叠板块来实现复杂布局。如需创建一个可折叠的内容块，需要为
容器添加 data-role="collapsible" 属性。在容器（div）内，添加一个标题元素（H1-H6），后跟您想要进行扩展的 HTML
标记：

一个默认展开的折叠板：

```html
<div data-role="collapsible" data-collapsed="false">
<h1>点击我 - 我可以折叠!</h1>
<p>I'm 现在我默认是展开的。</p>
</div>

```

一个默认折叠的折叠版：

```html
<div data-role="collapsible">
<h1>点击我 - 我可以折叠!</h1>
<p>我是可折叠的内容。</p>
</div>
```
<div data-role="collapsible">

我是可折叠的内容。

</div>
jMobile的列表使用非常有特色。可以在列表中添加图标。默认的列表用法如下：

```html
<ul data-role="listview">
<li><a href="#">列表项</a></li>
<li><a href="#">列表项</a></li>
<li><a href="#">列表项</a></li>
</ul>
```

列表样式的圆角和边缘，使用 data-inset="true" 属性设置
列表项也可以转化为列表分割项，用来组织列表，使列表项成组。
指定列表分割，给列表项 li 元素添加 data-role="list-divider" 属性即可。


jquery Mobile提供一个非常简单的方法，实现客户端搜索功能，筛选列表的选项。只需添加 data-filter="true" 属性即可:

```html
<ul data-role="listview" data-filter="true"></ul>
```

以上的介绍时关于jMobile的最基础用法，我们在下一篇将着重介绍表单基础和事件处理。