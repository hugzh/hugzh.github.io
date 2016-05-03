title: webpack学习实践系列(一)
date: 2016-05-02 19:59:34
categories:
    - 前端开发 
tags: 
    - 构建工具
    - 前端工程
---
Webpack是伴随着React出现的一个名词，近两年Webpack正在越来越受到前端开发者们的热捧。如果你已经使用过诸如Grunt、Gulp这样的工具，那么你对“构建”这个词肯定不会陌生。没错，Webpack就是当今最火的前端构建工具之一，配合Gulp使用可以搭建出非常强大的构建工具。<!--more-->趁着五一小假期，打算把最近自己折腾过的知识做个总结。这是上篇，主要是单纯的webpack使用，[下篇]()主要内容是结合Gulp尝试搭建更加强大的前端构建工具。

# 什么是Webpack？
按照[官方文档](http://webpack.github.io/docs/what-is-webpack.html)的解释，Webpack就是个模块打包工具，将模块及其依赖打包生成静态资源。在Webpack的机制里，所有的资源都是模块(js,css,图片等)，而且可以通过代码分隔([Code Splitting](http://webpack.github.io/docs/code-splitting.html))的方法异步加载，实现性能上的优化。

## Chunk的概念
chunk是使用Webpack过程中最重要的几个概念之一。在Webpack打包机制中，编译输出的文件包括entry（入口，可以是一个或者多个资源合并而成，由html通过script标签引入）和chunk（被entry所依赖的额外的代码块，同样可以包含一个或者多个文件）。从页面加速的角度来讲，我们应该尽可能将所有的js打包到一个bundle.js之中，但是总会有一些功能是使用过程中才会用到的。出于性能优化的需要，对于这部分资源我们可以做成按需加载，通过require.ensure方法实现:
```
require.ensure([], function(require) {
        var dialog = require('./components/dialog');
        // todo ...
    });
```
而固定的公用代码则独立打包到trunk之中。在Webpack的配置中，我们可以通过CommonsChunkPlugin插件对指定的chunks进行公共模块的提取。我们指定好生成文件的名字，以及想抽取哪些入口js文件的公共代码，webpack就会自动帮我们合并好。:
```
var chunks = Object.keys(entries);
plugins: [
    new  webpack.optimize.CommonsChunkPlugin({
        name: 'vendors', // 将公共模块提取，生成名为`vendors`的chunk
        chunks: chunks,
        minChunks: chunks.length // 提取所有entry共同依赖的模块
    })
],
```
## Loader
Loader就是资源转换器。由于在webpack里，所有的资源都是模块，不同资源都最终转化成js去处理。针对不同形式的资源采用不同的Loader去编译，这就是Loader的意义。Loader在使用之前必须先通过npm安装，然后在config里面通过module配置才能使用。举个例子:
```
module: {
    loaders: [{
        test: /\.(png|jpg|jpeg|gif)$/,
        loader: 'url?limit=10000&name=images/[name].[ext]'
    }]
}
```
上述配置中，test的作用是正则匹配，匹配到png或jpg或gif结尾的文件就采用url-loader来做对应的编译。由于loader都是默认以-loader后缀结尾的，所以可以省略后缀"-loader"，直接写成url。问号后面是参数，表示10000B以下的图片直接压缩成base64编码，超过10000B的图片输出到"images/文件名.拓展名"。上面的配置也可以这样写：
```
loaders: [{
    test: /\.(png|jpg|jpeg|gif)$/,
    loader: 'url-loader',
    query:{
        limit:'10000',
        name:'images/[name].[ext]'
    }
}]
```
配置中常用的loader：
* 处理样式，转成css，如：less-loader, sass-loader
* 图片处理，如: url-loader, file-loader。两个都必须用上。否则超过大小限制的图片无法生成到目标文件夹中
* 处理js，将es6或更高级的代码转成es5的代码。如： babel-loader，babel-preset-es2015，babel-preset-react
* 将js模块暴露到全局，使用expose-loader

## Plugin 
插件的引入和loader差不多，只是插件是以对象的形式引入。像静态资源路径的替换这种功能就能通过插件来处理。比如公用模块打包到trunk的插件：
```
var chunks = Object.keys(entries);
plugins: [
    new  webpack.optimize.CommonsChunkPlugin({
        name: 'vendors', // 将公共模块提取，生成名为`vendors`的chunk
        chunks: chunks,
        minChunks: chunks.length // 提取所有entry共同依赖的模块
    })
],
```
配置中常用的plugin：
* 代码热替换, HotModuleReplacementPlugin
* 生成html文件，HtmlWebpackPlugin
* 将css成生文件，而非内联，ExtractTextPlugin
* 报错但不退出webpack进程，NoErrorsPlugin
* 代码丑化，UglifyJsPlugin，开发过程中不建议打开
* 多个 html共用一个js文件(chunk)，可用CommonsChunkPlugin
* 清理文件夹，Clean
* 调用模块的别名ProvidePlugin，例如想在js中用$，如果通过webpack加载，需要将$与jQuery对应起来

# 搭建自己的构建集成环境
介绍完上面几个概念，我们就可以进入动手搭建脚手架的阶段了。
网上关于webpack的介绍文章不少，但是这些文章大多数是简略性的介绍或者是仅仅面向有一定webpack基础的开发者，对于完全从零开始的开发者来说理解起来有点吃力。这篇文章是完全面向webpack零基础的同学，根据我自己的填坑经历一步一步描述如何搭建一个简单的构建工具。
## 项目目录
假设我们要搭建的demo项目的目录结构是这样的：
```
- webapp/               # webapp根目录
  + node_modules        # node_modules
  - src/                # 开发目录
    - index             # index模块
      + images/         # webapp图片资源目录
      index.html        # 模板   
      index.js          # 模块entry
      style.less        # 样式表
  webpack.config.js     # webpack配置文件
  package.json          # 项目依赖文件
  config.js             # 项目配置文件
  README.md             # 项目说明
```

## 配置过程
项目目录确定之后，下面开始来着手配置webpack。
### 安装webpack
首先，全局安装webpack，至于怎么装...额，就不侮辱大家智商了。
### 安装依赖
这一步也非常简单，根据项目需要用到的依赖 npm i xxx--save-dev，也可以在配置webpack.config.js的过程中根据需要安装。
### 配置webpack
这一步基本是webpack配置的全部内容。由于webpack会默认读取根目录下的webpack.config.js文件，所以我们需要在根目录手动创建。
看看我们的webpack配置文件：
#### 1、首先，引入我们需要用到的npm模块
```
var path = require('path'); //node 原生path模块
var webpack = require('webpack'); // webpack
var glob = require('glob'); // glob模块，用于读取webpack入口目录文件
var ExtractTextPlugin = require('extract-text-webpack-plugin'); //webpack插件
var HtmlWebpackPlugin = require('html-webpack-plugin'); //webpack插件
var OpenBrowserPlugin = require('open-browser-webpack-plugin');//webpack插件
var CleanPlugin = require('clean-webpack-plugin')//webpack插件，用于清除目录文件
var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;//处理trunk
```
#### 2、读取入口文件
按照之前约定的项目目录，我们的webpack入口文件所在目录为src/index/index.js。所以，先要用glob读取模块入口。在这里，我们把读取目录定义为一个函数：
```
var getEntry = function() {
    var entry = {};
    //读取开发目录,并进行路径裁剪
    glob.sync('./src/**/*.js')
        .forEach(function(name) {
            var start = name.indexOf('src/') + 4,
                end = name.length - 3;
            var n = name.slice(start, end);
            n = n.slice(0, n.lastIndexOf('/'));
            //保存各个组件的入口
            entry[n] = name;
        });
    return entry;
};
```
然后将配置封装module.exports，定义入口entry，entry可以为字符串、对象或者数组，对应单页面和多页面应用：
```
module.exports = {
    entry: getEntry(),
    ...
}
```
#### 3、定义资源输出
资源打包输出的配置在output内，主要包括path、filename、chunkFilename以及publicPath。path是资源输出路径，filename是资源命名规则，chunkFilename是公共js打包后输出的命名，publicPath是静态资源的公共路径，比如线上CDN地址等，也可以不设置，这样CSS中的相对路径就不会包括publicPath。在output输出的时候可以根据开发环境或者生产环境选择不同的文件命名方法，因为一般来说，线上的资源都是要经过压缩的。比如我们定义一个prod变量判断当前编译环境:
```
    ...
    output: {
        path: path.resolve(__dirname, prod ? "./dist" : "./build"),
        filename: prod ? "js/[name].min.js" : "js/[name].js",
        chunkFilename: 'js/[name].chunk.js',
        publicPath: prod ? "http:cdn.mydomain.com" : ""
    },
    ...
```
[name]的值是根据入口entry显示的文件名。比如index.js这个入口文件，对应的output的[name]值就应该是“index”，当然，我们还可以根据需要使用[hash]、[id]这样的值。
#### 4、定义resolve
为了方便开发，我们可以定义自己的别名，以便很快捷地引用不同的模块，别名(alias)的定义是在resolve对象之中。比如，
```
resolve: {
    alias:{
        xyz: "/absolute/path/to/file.js" 
    }
}

```
那么，当我们在代码中require('xyz')的时候，实际上我们是引入'/absolute/path/to/file.js'这个文件。还可以配置extensions对象，使得开发过程中模块的处理可以忽略后缀。在我们的demo中，是这样配置的:
```
    ...
    resolve: {
        //配置项,设置忽略js后缀
        extensions: ['', '.js', '.less', '.css', '.png', '.jpg'],
        root: './src',
        // 模块别名
        alias: {}
    },
    ...
```
#### 5、配置loaders
loader的配置是在module对像中。根据文章开头部分的介绍，loader就是定义一个个资源处理器，demo项目主要用到下面几个loader：
```
    ...
    module: {
        loaders: [{
            test: /\.(png|jpg|jpeg|gif)$/,
            loader: 'url?limit=10000&name=images/[name].[ext]'
        }, {
            test: /\.less$/,
            loader: ExtractTextPlugin.extract('style', 'css!less')
        }, {
            test: /\.js[x]?$/,
            exclude: /node_modules/,
            loader: 'babel?presets[]=es2015&presets[]=react'
        }, {
            test: /\.html$/,
            loader: 'html?attrs=img:src img:srcset'
        }]
    },
    ...
```
ExtractTextPlugin.extract是用来提取出单独的CSS文件的插件，如果不使用这个插件处理样式文件，CSS会内联在页面中，不利于做样式表维护。babel用来做es6转换。
#### 6、定义Plugins
```
    ...
    plugins: [
        new HtmlWebpackPlugin({
            filename: 'index.html',
            template: './src/index/index.html'
        }),
        new CleanPlugin(['dist', 'build']),
        // 启动热替换
        new webpack.HotModuleReplacementPlugin(),
        new ExtractTextPlugin('[name].css', {
            allChunks: true
        }),
        new webpack.NoErrorsPlugin(),
        new OpenBrowserPlugin({
            url: 'http://localhost:8080'
        }),
        /* 公共库 */
        new CommonsChunkPlugin({
            name: 'vendors',
            minChunks: Infinity
        }),
    ]
    ...
```
HtmlWebpackPlugin插件用来自动在页面中注入chunk；HotModuleReplacementPlugin插件是用来做热替换的，每次开发环境下的资源发生变更之后都会自动重新打包输出，不需要重新构建；配置OpenBrowserPlugin插件可以在构建完成之后自动打开浏览器的localhost:8080路由；CommonsChunkPlugin插件定义chunk名字，文章开始部分已做详细介绍。
#### 7、定义webpack-dev-server
webpack-dev-server是webpack提供的静态资源服务器，它的存在使得开发可以脱离代理服务器。本地调试静态资源不再需要搭建本地服务器，简直是解救万民于水火呀！webpack-dev-server有多种配置形式，这里采用的是写死在config的方式，这种方式的特点是方便开发但是不灵活。
```
    ...
    devServer = {
        port: 8080,
        contentBase: './build', //定义静态服务器的基路径
        hot: true,
        historyApiFallback: true,
        publicPath: "",
        stats: {
            colors: true
        },
        plugins: [
        new webpack.HotModuleReplacementPlugin()
        ]
    }
    ...
```

#### 8、编译环境判断
在步骤3我们需要根据当前的编译环境来选择不同的资源输出方式。编译环境的判断可以通过定义node的script来设置。在我们项目根目录的package.json文件中，定义：
```
"scripts": {
    "dev": "webpack-dev-server",
    "build": "webpack",
    "deploy": "set NODE_ENV=production&&webpack -p --progress --colors"
  },
```
这样的话，终端执行"npm run dev" 就相当于执行 "webpack-dev-server"。如果执行"npm run deploy"，那就是编译生产环境，node就会设置环境变量"NODE_ENV"为"production"。然后在webpack的配置文件中，通过"process.env.NODE_ENV"就可以读取到"production"这个值。所以在配置的开头，我们这样定义一个局部变量:
```
var prod = process.env.NODE_ENV === 'production' ? true : false;
```
之后在配置文件的最后，根据当前的编译环境，如果是生产环境就配置压缩丑化插件"UglifyJsPlugin"，如果是开发环境就配置webpack-dev-server：
```
// 判断开发环境还是生产环境,添加uglify等插件
if (process.env.NODE_ENV === 'production') {
    module.exports.plugins = (module.exports.plugins || [])
        .concat([
            new webpack.DefinePlugin({
                __DEV__: JSON.stringify(JSON.parse(process.env.DEBUG || 'false'))
            }),
            new webpack.optimize.UglifyJsPlugin({
                compress: {
                    warnings: false
                }
            }),
            new webpack.optimize.OccurenceOrderPlugin(),
        ]);
} else {
    module.exports.devtool = 'source-map';
    module.exports.devServer = {
        port: 8080,
        contentBase: './build',
        hot: true,
        historyApiFallback: true,
        publicPath: "",
        stats: {
            colors: true
        },
        plugins: [
        new webpack.HotModuleReplacementPlugin()
        ]
    };
}
```

## 模块代码
至此，我们已经完成了一个基本的webpack配置，接下来就是使用webpack构建代码了。我们定义的index模块的代码入口是index.js，在Index.js内通过require()方式引入不同的资源用于打包：
```
require('./style.less'); //引入.less预处理文件
require('./index.html'); //引入同级目录的index.html文件
```
然后，执行"npm run dev"即可查看本地环境的静态资源效果。demo项目的完整代码可以[查看github](https://github.com/hugzh/webpack-simple-demo)

# 总结
webpack的零碎知识点还有很多，这里只是根据自己的学习心得搭建出来的简单构建环境。目前的构建还是不够灵活强大的，后面的时间我会继续探索结合gulp定制的webpack构建工具，在当前构建的基础上搭建一套更加完善的构建工具。

本文出自橙乡果汁的博客([blog.hugzh.com](http://blog.hugzh.com/2016/05/02/webpack%E5%AD%A6%E4%B9%A0%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97-%E4%B8%80/))，转载请注明出处!
