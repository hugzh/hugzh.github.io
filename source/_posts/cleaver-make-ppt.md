title: 如何使用Cleaver制作html5幻灯
tags:
  - cleaver
  - 幻灯
categories:
  - 前端开发
date: 2015-07-07 23:24:02
---

Cleaver是一款基于html5的插件，依赖于Markdown文档格式，可以用来快速地制作幻灯片。这里主要是讲一讲如何安装和使用Cleaver制作炫酷幻灯片。
<!--more-->
### **1.Cleaver安装**

通过nodejs来安装Cleaver，而且只支持Linux系统，windows下暂时不支持。

进入新建好的文件夹demo，使用命令 npm install cleaver -g ，即可安装完成。

### **2.<span style="color: #000000;">Cleaver基本使用</span> **

执行cleaver命令，解析一个markdown文件。用cleaver自带的例子：

` cleaver node_modules/cleaver/examples/basic.md`

basic.md的内容是这样的：

```javascript
title: Basic Example
author:
name: Jordan Scales
twitter: jdan
url: http://jordanscales.com
output: basic.html

--

# Cleaver 101
## A first look at quick HTML presentations

--

### A textual example

Content can be written in **Markdown!** New lines no longer need two angle brackets.

This will be in a separate paragraph.

<img src="http://whatismarkdown.com/workspace/img/logo.gif" alt="Drawing" style="width: 150px;"/>

![markdown-logo](logo.gif)
<img src="logo.gif" />

[Here's a link](http://google.com).

--

### A list of things

* Item 1
* Item B
* Item gamma

No need for multiple templates! [Another link](http://google.com).

--

### Unicode

* 林花謝了春紅 太匆匆
* 胭脂淚 留人醉 幾時重
* Matching Pairs «»‹› “”‘’「」〈〉《》〔〕
* Greek αβγδ εζηθ ικλμ νξοπ ρςτυ φχψω
* currency ¤ $ ¢ € ₠ £ ¥

--

### A code example

```javascript
// cool looking code
var func = function (arg1) {
return function (arg2) {
return "arg1: " + arg1 + "arg2: " + arg2;
};
};

console.log(func(1)(2)); // result is threeAnd here is some `inline code` to check out.

```

&nbsp;

然后打开该目录下的basic.html即可查看幻灯

[![](http://bloghugzh-wordpress.stor.sinaapp.com/uploads/2015/07/cleaver-firefox-300x217.png "cleaver-firefox")](http://bloghugzh-wordpress.stor.sinaapp.com/uploads/2015/07/cleaver-firefox.png)

### **3.简单做一些配置：**

1). title: HTML的标题名

2). author: 作者信息,在最后一页显示

包括下面信息：

*   name: 名字
*   url: 个人网站
*   twitter: 微博
*   email: 邮件地址
3). theme: 皮肤(theme: jdan/cleaver-retro)

4). style: css样式表(style: css/main.css)

5). output: 生成的HTML文件名(output: basic.html)

6). controls: 控制按钮(controls: true)

7). progress: 顶部显示进程条(progress: true)

8). agenda: 生成一个目录页(progress: false)

9). encoding: 文档的字符编码(encoding: utf-8)

10). template: 设置每张slide的模板(template: template/slide.mustache)

11). layout: 设置HTML模板(template: template/layout.mustache)

### **4.正文**

正文必须使用Markdown语法。

&nbsp;