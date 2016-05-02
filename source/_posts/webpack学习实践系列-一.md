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

## Trunk的概念
Trunk是使用Webpack过程中最重要的几个概念之一。在Webpack打包机制中，编译输出的文件包括entry（入口，可以是一个或者多个资源合并而成，由html通过script标签引入）和chunk（被entry所依赖的额外的代码块，同样可以包含一个或者多个文件）。从页面加速的角度来讲，我们应该尽可能将所有的js打包到一个bundle.js之中，但是总会有一些功能是使用过程中才会用到的。出于性能优化的需要，对于这部分资源我们可以做成按需加载，通过require.ensure方法实现:
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
介绍完上面几个概念，我们就可以进入动手搭建脚手架的阶段了。(待续)