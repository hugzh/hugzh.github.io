title: 使用nodejs做一个新闻网页爬虫——Nodejs初体验
id: 71
categories:
  - javascript
  - Nodejs
date: 2015-04-16 20:45:12
tags:
---

昨天在leader的要求下做了一个小小的作业，作为近期项目的一个入门练习。要写的是一个网页爬虫，使用node来爬取网易新闻页 http://news.163.com的首页所有的新闻连接，存入urls.txt文件。然后从urls.txt文件中选取一条url进行新闻内容的爬取，存入news.txt文件中。

本来对nodejs的了解仅仅停留在“hello world”的水平，虽然很早就曾打算学习这们传说中无比流弊的语言，无奈最近因为找实习各种事务缠身，始终没时间深入研究node。好在leader给我们的任务恰恰成为我学习node的推动力，哈哈。

回到刚才说的任务，网页爬虫。既然是对网页内容的提取，首先必须得搭建好一个良好的架构。主要是两个方面：一个是对整个页面的爬取，另一个就是对页面关键词的提取。爬取页面肯定要用到node的http模块，在这里我是用了http.get的方法。对于页面元素的提取，我是用了最近刚刚接触过的cheerio这样一个库，这是一个非常轻量型的第三方插件，它的使用语法和jQuery几乎完全一样。比如选择器的语法：

$(selectior,[context],[root])

选择器在 Context 范围内搜索，Context又在Root范围内搜索。selector 和context可是一个字符串表达式，DOM元素，和DOM元素的数组，或者chreeio对象。root 是通常是HTML 文档字符串。

```javascript
$('.apple', '#fruits').text()
//=&gt; Apple

$('ul .pear').attr('class')
//=&gt; pear

$('li[class=orange]').html()
//=&gt; &lt;li class="orange"&gt;Orange&lt;/li&gt;
```

又比如获取html元素内容的语法：.text( [textString] )

```javascript
$('.orange').text()
//=&gt; Orange

$('ul').text()
//=&gt;  Apple
//    Orange
//    Pear</pre>

```

下面是我整个代码的目录结构

[![](http://bloghugzh-wordpress.stor.sinaapp.com/uploads/2015/04/捕获-300x112.jpg "捕获")](http://bloghugzh-wordpress.stor.sinaapp.com/uploads/2015/04/捕获.jpg)

其中app.js就是负责爬取新闻列表页的url，而getNew.js是负责对新闻内容的抓取。这个项目的难点就是中文编码的转码问题，由于网易新闻主页的html编码方式是GBK，而Nodejs本身是不支持GBK字符的读写的，所以抓取下来的新闻内容就会出现乱码。这是Nodejs入门的一个经典案例，花了我很长时间来解决。后来查找资料得知使用iconv插件来进行转码，最终解决了这个烦人的问题。代码如下：

```javascript
var cheerio = require('cheerio')
var request = require('request')
var fs = require('fs')
var iconv = require('iconv-lite')
var http = require('http')

/*get one url from file urls.txt*/
fs.readFile('urls.txt','utf-8',function (err,data) {
	if(err){
		console.log(err);
		return ;
	}

	var urlArray = data.split('\"},');
	var urlstr = urlArray[0].split(':\"');

	var url = urlstr[1];
	console.log(url);
	http.get(url,function (res) {
		//this step is very important
		//it change gbk unicode into binary,avoid error

		res.setEncoding('binary');
		var article = '';
		res.on('data',function (data) {
			article += data;
		}).on('end', function () {
			var buf = new Buffer(article, 'binary');

			//decode the content in gbk unicode
			//depend on plus-in iconv
			var gbkStr = iconv.decode(buf, 'GBK');

			var $ = cheerio.load(gbkStr);
			var result = [];
			var title = $('#h1title').text();
			var cont  = '';
			$('p','#endText').each(function(index,ele){
				cont+=$(this).text();
			});

			result.push({ArticleTitle:title,ArticleContent:cont});
			var file = 'article.txt';
			fs.writeFile(file,JSON.stringify(result),function(err){
				if(err){
					console.log("write file failed"+err);
					return ;
				}
			});

		})
		}).on('error', function (err) {
			console.log(err);
		});
});

```

这就是整个项目的核心代码所在。如果要想查看app.js是如何写的，可以查看我的github地址，https://github.com/hugzh/node。这里就不贴代码了。