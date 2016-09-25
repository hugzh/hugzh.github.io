title: 基于yeoman定制的交互式命令行脚手架
date: 2016-09-25 14:07:19
tags:
    - 构建工具
categories:
    - Nodejs
    - 前端开发
---
脚手架这个词估计做前端的都很熟悉。在没有实现前端工程化的年代，前端代码的组织都是纯手工维护的。比如我要做一个网站页面，那么我需要手动创建一个文件夹来存放代码文件，我把它命名为demo。然后在demo目录下创建src文件夹，在src文件夹内创建css文件夹、js文件夹、image文件夹、lib文件夹等等...一切都是手工维护<!-- more -->。自从node.js出现后，前端开发才慢慢开始告别刀耕火种，越来越多的自动化工具充斥我们的眼球。模板生成、代码压缩、构建打包、自动部署...这些已经成为构建前端工程项目的标配。那么，一个模板生成的命令行工具的原理是什么？怎样开发一个属于自己的命令行脚手架工具？希望我写的这篇小文章会给大家带来一点启发。

## 原理
生成模板文件的方式可以是本地新建空白文件，然后进行文件内容读写；又或者是把本地已有的模板进行配置信息填充。然而我们知道，IO读写的速度非常慢，性能消耗大。但是一个模板生成器(Generator)如果是基于已有的模板文件进行配置填充，然后在copy到项目目录对应的位置，那会比直接读写磁盘效率更高。所以一般来说，模板生成器会采用第二种工作原理。

## Yeoman-generator
模板生成器的脚手架有很多，前端领域每天都会有很多类似的轮子源源不断地从开源社区流出。这里我用来开发自己的generator的工具是[Yeoman](http://yeoman.io/)。Yeoman的Logo是一个戴着红帽子的大胡子，它是一个通用的脚手架搭建系统，可以创建任何的类型的app。同时它又是"语言无感知"的，支持创建任何类型开发语言的项目，Web, Java, Python, C# 等等。Yeoman的通用性在于，它本身不做任何决定，所有的操作都是通过Yeoman环境里面的各种generator实现的。通过自定义generator，我们可以创建任何格式的项目目录。这是Yeoman的最大魅力之处。另外，Yeoman通过提供promting这个方法实现输入式命令行交互，可以让用户自由填写配置信息，交互体验也非常棒。下面说说怎样基于Yeoman开发一个简单的generator：

### Simple-dir
simple-dir是我自己捣鼓的一个很简单的Yeoman-generator，在这里我拿它来作为讲解示例，大家也可以打开[详细代码](https://github.com/hugzh/generator-simple-dir)来看，欢迎star，也欢迎提issue。
### 第一步，package.json
开发一个Yeoman-generator，我们要做的第一步就是配置package.json。有几个关键的地方，一个是，name的值的格式必须是"generator-"前缀 + Yeoman-generators官方源列表上的唯一值（如果你要共享你的generator到官方generator源的话）；第二个就是，keywords属性必须包括"yeoman-generator"这个值；第三，files属性是命令自定义文件,app是默认的命令；第四，必须要安装最新版本的yeoman-generator依赖，可以直接运行：npm install --save yeoman-generator 获取最新的版本号。详细的package.json可以看下面这份：

```
{
  "name": "generator-simple-dir",
  "version": "0.0.1",
  "description": "A very simple template generator",
  "files": [
    "generators/app",
    "generators/comp",
    "generators/page"
  ],
  "author": "橙乡果汁",
  "license": "MIT",
  "keywords": [
    "yeoman-generator"
  ],
  "repository": {
    "type": "git",
    "url": "git@github.com:hugzh/generator-simple-dir.git"
  },
  "bugs": {
    "url": "https://github.com/hugzh/generator-simple-dir/issues"
  },
  "dependencies": {
    "glob": "^7.1.0",
    "mkdirp": "^0.5.1",
    "yeoman-generator": "^0.24.1"
  }
}
```

对应的src目录格式应该是这样的：

> ├───package.json
> └───generators/
>     ├───app/
>     │   └───index.js
>     ├───comp/
>     │   └───index.js
>     └───page/
>         └───index.js

你也可以直接把files属性直接写成：

```
"files": [
  "app",
  "comp",
  "page"
]

```

但是这样的话，你的代码根目录就必须直接包含app,comp和page文件夹。

### 第二步，拓展generator
这里我们有三个generator——app,comp和page。以page为例，我们来实现一个generator。

首先，需要继承Yeoman提供的generator基类:

```
var generators = require('yeoman-generator');
module.exports = generators.Base.extend();

```

然后我们就可以在基类内部重写generator的方法了。Yeoman提供了一系列的基类方法：

> initializing - 初始化 (检查当前项目状态、获取配置文件内容等等)
> prompting - 获取用户输入，实现与用户的交互 (通过this.prompt()调用)
> configuring - 保存配置并配置整个项目 (比如创建 .editorconfig 文件和其他媒介文件)
> default - 当定义的方法没有匹配任何基类方法的时候用到
> writing - 根据自定义的规则写入具体的generator文件 (routes, controllers, etc)
> conflicts - 内部冲突处理
> install - 安装npm、bower等依赖的地方
> end - 在最后调用, 实现cleanup, say good bye等功能。

在示例[generator-simple-dir](https://github.com/hugzh/generator-simple-dir)里，page这个generator的作用是创建页面，需要生成html/css/js文件。在generators.Base.extend函数内部，page实现了 initializing、prompting、writing、end这几个方法。对于prompting这样的异步方法，需要在交互结束的时候调用this.async()来结束异步任务。Yeoman实现用户交互的核心方法是prompting，它是一个异步的方法，并且返回一个promise。prompting方法通过一个数组参数，可以实现链式的用户输入。其中input类型的是用户输入自定义内容，confirm类型是作为True/False判断的prompt，输入Y/N。官方的示例如下：

```
module.exports = generators.Base.extend({
  prompting: function () {
    return this.prompt([{
      type    : 'input',
      name    : 'name',
      message : 'Your project name',
      default : this.appname // Default to current folder name
    }, {
      type    : 'confirm',
      name    : 'cool',
      message : 'Would you like to enable the Cool feature?'
    }]).then(function (answers) {
      this.log('app name', answers.name);
      this.log('cool feature', answers.cool);
    }.bind(this));
  }
})
```

如果你想要记住用户输入的一个内容，用来做后面输入的默认值的话，还可以通过增加store:true配置来实现。
在generator-simple-dir里面，page这个generator包含4个执行步骤：初始化、获取用户输入、根据用户输入生产模板文件、结束返回，实现的代码如下：

```
'use strict';
var generators = require('yeoman-generator');
var glob = require('glob');

module.exports = generators.Base.extend({
  // init
  initializing: function() {
    this.existedFile = [];
    this.pageName = '';
    // 遍历./pages
    var pageFiles = glob.sync(this.destinationPath('./pages/*/'));
    var reg = /\/(\w+)(\/$)/;
    pageFiles.forEach(function(v) {
      if (v && v.lastIndexOf('/') > -1) {
        this.existedFile.push(reg.exec(v)[1]);
      }
    }.bind(this));
  },

  prompting: function() {
    var done = this.async();
    var promptConf = [{
      type: 'input',
      name: 'pageName',
      message: '请输入页面名称:',
      default: 'page_demo',
      // 校验page是否已存在
      validate: function(input) {
        if (this.existedFile && this.existedFile.indexOf(input) >
          -1) {
          this.log('页面已存在，请换一个页面名称！');
          return false;
        } else {
          return true;
        }
      }.bind(this)
    }, {
      type: 'input',
      name: 'pageTitle',
      message: '页面Title描述：',
      default: 'Title'
    }, {
      type: 'confirm',
      name: 'isNeedStyle',
      message: '是否需要样式表?',
      default: true
    }, {
      type: 'confirm',
      name: 'isPc',
      message: '是否PC端的页面？',
      default: false
    }];

    return this.prompt(promptConf)
      .then(function(props) {
        this.pageName = props.pageName;
        this.pageTitle = props.pageTitle;
        this.isNeedStyle = props.isNeedStyle;
        this.isPc = props.isPc;

        done();
      }.bind(this));
  },
  writing: function() {
    var tplArr = ['page.html', 'page.js', 'page.css'];
    var pageConf = {
      pageName: this.pageName,
      pageTitle: this.pageTitle,
      isNeedStyle: this.isNeedStyle,
    };
    if (this.isPc) {
      tplArr[0] = 'page.pc.html';
    }
    if (!this.isNeedStyle) {
      tplArr.pop();
    }

    tplArr.forEach(function(value, index) {
      // (from,to,content)
      this.fs.copyTpl(
        this.templatePath(value),
        this.destinationPath('pages/' + pageConf.pageName + '/' + pageConf.pageName +
          '.' + value.split(
            '.').pop()),
        pageConf
      );
    }.bind(this));

  },
  end: function() {
    this.log('新建页面完成!')
  }
});
```

### 定制模板
prompting方法是用来获取用户输入，writing方法是根据用户输入内容生成模板文件。之前说到，模板生成器的一般原理是用获取的配置信息渲染好模板，再拷贝到项目目录对应的位置。所以，在writing方法里面，需要实现模板渲染和拷贝。在Yeoman-generator里，需要的模板文件默认放在templates文件夹里，所有文件相关的操作通过[this.fs对象](https://github.com/sboudrias/mem-fs-editor)来实现。this.fs.copyTpl就是我们用来拷贝渲染好的模板文件的方法，需要输入三个参数：模板源路径、需要拷贝到的项目路径、模板渲染内容对象。模板的渲染是基于ejs模板引擎的语法。根据我们定义的项目结构，page的实现如下：

```
this.fs.copyTpl(
  this.templatePath(value),
  this.destinationPath('pages/' + pageConf.pageName + '/' + pageConf.pageName +
    '.' + value.split(
      '.').pop()),
  pageConf
);

```

更详细的Yeoman-generator文件操作文档请点击[这里](http://yeoman.io/authoring/file-system.html)。

## commander
上面我们讲解了Yeoman-generator的定制，也展示了一个简单的generator——"simple-dir"。为了把simple-dir很优雅地跑起来，我们需要搞一个命令行工具。基于Nodejs开发自己的命令行工具是很简单的事情，因为TJ大神已经为我们贡献了屌炸天的工具——commander.js。关于commander的使用教程有很多，也比较容易上手，如果你还没有了解过commander.js，推荐阅读这两篇文章：[《Commander：node.js命令行接口的完全解决方案》](http://hao.jobbole.com/commander-js/)和[《Commander写自己的Nodejs命令》](http://blog.fens.me/nodejs-commander/)。

有了commander的基础之后，我们将Yeoman-generator封装到自定义好的命令中。比如我已经封装好了自己的命令行工具，它的名字叫做atdir(取自auto director)，我们想要实现只需要运行 "atdir page" 就会自动生成需要的 html/css/js。然后我们只需要在atdir里面定义page.js：

```
module.exports = function() {
  var yeoman = require('yeoman-environment');
  var env = yeoman.createEnv();
  env.lookup(function() {
    env.run('simple-dir:page', {
      'skip-install': true
    }, function(err) {
      if (err) {
        throw err;
      }
    });
  });
}

```
env.lookup()的作用是遍历用户机器上安装好的generator，接入到Yeoman-environment，比如我们simple-dir的init、page或者comp命令。然后运行env.run()。由于我已经将simple-dir发布到npm包了，所以可以直接调用env.run('simple-dir:page'，function（）{}）。如果你不想将generator发布到npm，然后又想在本地使用generator的话也可以，直接进入generator的根目录，执行npm link，simple-dir 指令就会关联到本地的npm里面，Yeoman就能找到 "simple-dir:page" 这个指令啦！

## 小工具——atdir
atdir就是上面说的命令行小工具，想要了解命令行的详细封装方法可以[戳这里](https://github.com/hugzh/atdir)。由于atdir没有发布到npm源，不能直接npm i。如果想要运行起来的话，请先把atdir源码clone到本地，进入到atdir根目录，执行npm link，npm install 之后就可以愉快的执行atdir命令啦~~~
附上几张运行界面截图:
$ atdir init
![atdir init](http://p1.bpimg.com/575150/b729cf88363ad5db.png)

$ atdir page
![atdir page](http://p1.bpimg.com/575150/10d75b6287edabd0.png)

$ atdir comp
![atdir comp](http://p1.bpimg.com/575150/ac10496e5496593a.png)

## 结语
这只是Yeoman-generator的简单用法，意图在于学习搭建一个自己的命令行脚手架，其实还有很多可以完善的地方，比如目前的模板目录是固定的，可以考虑实现更灵活的配置；还可以加上webpack等打包工具的config实现自动构建等等，这个就留到后面再去拓展。大家有什么想法也可以在github上提issue，欢迎指正!