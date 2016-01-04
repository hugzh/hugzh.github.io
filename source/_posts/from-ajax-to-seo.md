title: 从AJAX的使用到爬虫友好到SEO优化
tags:
  - SEO
  - 爬虫
id: 81
categories:
  - Nodejs
date: 2015-05-14 21:26:09
---

&nbsp;

#  关于Ajax的使用注意

最近做爬虫项目的时候发现一个比较值得探讨的问题：由于Ajax的无刷新加载特性，web开发者都比较喜欢使用ajax进行与服务器的数据交互。很多DOM元素都是在基本DOM文档加载完毕之后再进行加载的。这种做法虽然对用户非常友好，但是有时候对于程序（搜索引擎）来说，页面信息获取也会带来一些挑战。
<!--more-->
## 1.  关于异步加载
Ajax的使用是非常广泛的。作为前端开发者，我们往往喜欢JSON数据这种交互模式,所以在渲染页面的时候也倾向于使用ajax从服务器获取JSON数据，从而实现无刷新渲染。这么做对于用户体验肯定是加分的。但是大规模地使用Ajax有时候会使得爬虫很难去获得想要的数据，当然如果我们设计的网站本身就是不想要别人抓取到数据的那就另当别论（除此之外还要加上一些JS动态加载数据以及数据编码等开发方式）。

我们知道，所谓的网络“爬虫” ，就是通过一些http的request模块获取某个URL对应的页面的整个html文档，然后根据一些XPath或者CSS路径语法提取出文档的DOM节点内容，从而获取想要的数据。因此，如果这个页面的重要数据是通过JS执行Ajax来实现的话，就要求爬虫自己去模拟JS事件的触发（这种情况出现最多的就是”load more“这种情景）。甚至有时候如果封装的函数和要传输的参数太过复杂的话，爬虫根本无法通过模拟事件获取最新数据。这样无形之中就会使得爬虫无法抓取更多的网页数据。也就是对爬虫不友好。当然，很多脚本语言已经实现了很多对于事件模拟的模块，但是如果在不是非常必要的情况下，我觉得还是不要滥用Ajax。 那么，爬虫在获取JSON数据的时候具体是怎样工作的呢？我这里举一个处理样例，这个例子是基于Nodejs的Osmosis插件做的爬虫处理。
首先  ，我们先来看看这个AJAX的访问url "http://russia.tv/video/json/menu_id/267?page=4" ,访问这个URL得到的数据格式如图：

[![](http://bloghugzh-wordpress.stor.sinaapp.com/uploads/2015/05/捕获-300x89.jpg "捕获JSON")](http://bloghugzh-wordpress.stor.sinaapp.com/uploads/2015/05/捕获.jpg)

我们需要的仅仅是里面的'url'键对应的值，所以爬虫可以这样去提取数据：
```
    var json_text = JSON.parse(data.json);
    // get flower
    // {"last_page":0,"row_list":[{"url":'xxxxx'}]}
    var urls = [];
     _.each(json_text.row_list, function (url) {

         urls.push({
            url: url.url
        });
    });
```
## 2.搜索引擎优化(SEO)

既然聊到了爬虫，那么第二个我想说的问题就是关于SEO。爬虫只是人工搜索数据的一种手段，但是更高级别的大规模数据搜索往往是谷歌百度等搜索引擎。这类的搜索引擎是使用robots来进行数据挖掘的。这就要求前端开发者需要做一些页面优化，使得自己的站点可以更好的被搜索引擎发现。这就是SEO的主要内容，比较简单的SEO方法就是添加meta标签。meta是用来在HTML文档中模拟HTTP协议的响应头报文。META标签是HTML语言HEAD区的一个辅助性标签，它位于HTML文档头部的&lt;HEAD&gt;标记和&lt;TITLE&gt;标记之间，它提供用户不可见的信息。以下是几种meta标签的用法：
<div align="left">1、&lt;meta name="Generator" content=""&gt;用以说明生成工具（如Microsoft FrontPage 4.0）等；</div>
<div align="left">2、&lt;meta name="KEYWords"  content=""&gt;向搜索引擎说明你的网页的关键词；</div>
<div align="left">3、&lt;meta name="DEscription"  content=""&gt;告诉搜索引擎你的站点的主要内容；</div>
<div align="left">4、&lt;meta name="Author"  content="你的姓名"&gt;告诉搜索引擎你的站点的制作的作者；</div>
<div align="left">5、&lt;meta name="Robots"  content= "all|none|index|noindex|follow|nofollow"&gt;</div>
<div align="left">　 其中的属性说明如下：</div>
<div align="left">　 设定为all：文件将被检索，且页面上的链接可以被查询；</div>
<div align="left">   设定为none：文件将不被检索，且页面上的链接不可以被查询；</div>
<div align="left">   设定为index：文件将被检索；</div>
<div align="left">   设定为follow：页面上的链接可以被查询；</div>
<div align="left">   设定为noindex：文件将不被检索，但页面上的链接可以被查询；</div>
<div align="left">   设定为nofollow：文件将不被检索，页面上的链接可以被查询。</div>
<div align="left"> 还有一种比较常见的meta标签是用户社交（**Open Graph Protocol**）。我们可以发现很多网页的head里面包含&lt;meta property="og:title" content="xxx"&gt;这样的标签。</div>
<div align="left">Meta Property=og标签是什么呢?og是一种新的HTTP头部标记，即Open Graph Protocol。即这种协议可以让网页成为一个“富媒体对象”。</div>

用了Meta Property=og标签，就是你同意了网页内容可以被其他社会化网站引用等，目前这种协议被SNS网站如Fackbook、renren采用。SNS已经成为网络上的一大热门应用，优质的内容通过分享在好友间迅速传播。为了提高站外内容的传播效率，2010年F8会议上Facebook公布 了一套开放内容协议(Open Graph Protocol)，任何网页只要遵守该协议，SNS就能从页面上提取最有效的信息并呈现给用户。
