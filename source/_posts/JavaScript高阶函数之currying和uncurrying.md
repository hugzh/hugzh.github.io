title: JavaScript高阶函数之currying和uncurrying
date: 2016-01-26 17:27:49
tags:
    - js
    - curring
    - uncurrying
categories:
    - 前端开发
---
在JavaScript高阶函数的应用里面，柯里化 （currying）和反柯里化（uncurrying）是两组比较有意思的应用例子。
## 一、函数柯里化（function currying）
### 1、什么是柯里化
柯里化，俗称“部分求值”。一个柯里化函数首先是会接受一些参数，但是接受这些参数之后，该函数并不会立即求值，而是继续返回另一个函数<!--more-->，刚才传入的参数在函数形成的闭包中被保存起来。等到函数被真正需要求值的时候，之前传入的所有参数会被一次性用于求值(注：摘自AlloyTeam出版的《JavaScript设计模式与开发实践》)。也就是说，柯里化是函数式编程的一种技巧，用于把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数。
听起来是比较抽象的，下面就拿一个具体的函数柯里化实际场景来做例子：
假设有这样一个需求：小明去某著名旅游区玩，想吃一下小龙虾。小明是个吃货，一两只虾肯定是不够塞牙缝的，又不知道要买多少才能填饱胃口。于是打算先吃饱再结账。也就是说，小明在这个过程中不关心吃了多少虾，只关心吃饱了之后结账银行卡要减多少数字。这就是柯里化的场景！餐厅老板可能会选择使用下面的算法来结算：
```
var money = 0;
var pay = function(oneLobster) {
    money+=oneLobster;
}
pay(38);
pay(38);
pay(38);
console.log(money)
// 小明吃了三只虾，money=114
```
上面的代码虽然也可以计算这次吃虾要花多少，但是小明每吃一次都要计算一次总价格，岂不是要累死？于是老板想要一个通用的curry函数，用来计算小明吃饱之后买单要花多少，函数如下：
```
var curry = function (fn) {
    var args = [];
    return function() {
        if (arguments.length === 0) {
            return fn.apply(this, args);
        } else {
            args = args.concat([].slice.call(arguments));
        }
    }
};
```
接下来就是用这个函数柯里化小明吃虾的函数了：
```
var pay = (function(){
    var money = 0;
    return function(){
        for (var i=0,l=arguments.length;i <l; i++) {
            money+=arguments[i];
        }
        return money; 
    }
})();

var pay = curry(pay)
pay(38);
pay(38);
pay(38);
pay(38);
... //吃饱买单!
pay(); 
```
最终小明吃了两盘虾，叫老板来结账。老板说：“上帝你好，您一共消费RMB 760块！”
“What?我才吃了两盘，怎么算都是76块钱嘛。欺负我没读过书？”小明愤怒道。
“不好意思，我们的虾来自火星，38块钱一只，您吃了两盘，一共20只”老板一边诡辩一边拿出餐牌，上面只标记“来自星星的虾 38”。
小明当场吐血！！！
...
好吧，以上就是函数柯里化的一个应用场景，当调用pay()的时候，如果明确带上一个参数，那么本次不进行真正的求值计算，只是保存参数（每吃一只虾先记账），并返回一个函数。只有当老板要结算的时候调用不带参数的pay()，才会正真利用前面保存的所有参数进行计算求值（吃完虾结账）。
但是，从买虾的例子来看，函数柯里化和第一个不使用柯里化的方案看起来并没有什么高端的地方，反而使得代码更复杂。那为什么要介绍柯里化呢？
### 2、柯里化的作用
柯里化的应用之一就是惰性加载函数（也有人叫“提前返回”）。很常见的一个例子就是跨浏览器的事件注册，为了兼容ie它老人家，我们一般会这么写事件注册：
```
var addEvent = function(ele, type, fn) {
    if (window.addEventListener) {
        return ele.addEventListener(type,fn,false);
    } else if (window.attachEvent) {
        return ele.attachEvent(type, fn);
    }
};
```
那这么写的问题在哪里呢？我们每次使用addEvent为元素添加事件的时候，都会走一遍if...else if ...分支，其实只要一次判定就可以了，这就是惰性函数的用处了。此时函数在第一次进入分支之后，会在函数内部重写这个函数，重写的这个函数就是我们需要的addEvent函数了，下一次进入addEvent函数的时候，函数不再存在条件分支。详细实现如下：
```
var addEvent = function(ele, type, fn) {
    if (window.addEventListener) {
        addEvent = function(ele, type, fn) {
            ele.addEventListener(type,fn,false);
        } 

    } else if (window.attachEvent) {
        addEvent = function(ele, type, fn) {
            ele.attachEvent(type,fn);
        } 
    }
    //执行
    addEvent(ele, type, fn);
}
```
## 二、反柯里化(uncurrying)
个人觉得，其实“反柯里化”这种译法不是特别的贴切。从字面上看，它的作用好像是和柯里化的作用相反，事实上两者之间并没有存在这种关系。
Uncurrying的话题来自JavaScript之父 Brendan Eich在2011年发表的一篇推特，它解决的问题是让对象去借用一个原本不属于自己的方法。在JS的语言环境中，我们可以通过call和apply完成this的转化，同样，用uncurrying可以解决this的转化问题。
uncurrying的实现非常地简单：
```
Function.prototype.uncurrying = function() {
    var self = this; //保存现场
    return function() {
        //截去第一个参数
        var obj = Array.prototype.shift.call(arguments);
        return self.apply(obj,arguments);
    };
};
```
这段代码的作用就是把属于数组原型对象的函数转化为一个普通的函数，实现相同的功能。比如说，实现一个slice()函数：
```
var slice = Array.prototype.slice.uncurrying();
var a = [1,2,3];
console.log(slice(a,1));// [2,3]
```
这样，我们就可以把很长的一坨Array.prototype.slice.call()转化为普通的slice函数给自己定义的对象使用，使得代码更加简洁明了。
Uncurrying的另一种实现方式：
```
Function.prototype.uncurrying = function() {
    var self = this;
    return function() {
        return Function.prototype.call.apply(self, arguments);
    };
};
```
## 结尾
函数柯里化和反柯里化是高阶函数运用中比较有意思的技巧，在进行js代码编写的时候恰当地使用高阶函数可以使代码性能得到很大的提升。目前这些技巧在JavaScript的各种设计模式里面得到很好地使用。

参考资料：
《JavaScript设计模式与开发实践》
[JS中的柯里化(currying)](http://www.zhangxinxu.com/wordpress/2013/02/js-currying/)
[JavaScript中有趣的反柯里化](http://web.jobbole.com/32059/)
