title: 单元测试工具Mocha和SuperTest组合
date: 2015-12-08 20:07:57
tags:
    - mocha
    - node
categories:
    - Nodejs
---

毕业季，一堆乱七八糟的事情搞得已经整整两个月没有时间写写笔记了。最近进行着一个Node小项目，其中关于单元测试部分花了不少时间去踩坑。所以在这里简单总结一下关于Node开发测试框架Mocha(摩卡)的一些基础用法。由于本人是入门级别的Node开发者，所以这篇文章也是仅适用于和我一样的初级Node开发者，如有不正确的地方欢迎大神们指正！

## 一、Mocha
Mocha是非常流行JavaScript测试框架之一，在浏览器和Node环境都可以使用。这里主要是针对Node环境。
<!--more-->
### 1、安装
关于Mocha的安装当然是万能的npm了，直接npm install就OK，这里不再罗嗦了。
### 2、使用
然后就是Mocha的语法了。首先来看一下最简单例子：
```
// 这是一个简单的加法函数
//add.js
function add(a, b){
    return a+b;
}
module.exports = add;
```
现在用Mocha写一个测试脚本去测试这个函数。一般来说，测试脚本的命名要和测试的模块一致，后缀为.test.js。这里测试脚本的名称为add.test.js，表示对add模块进行测试。
```
//add.test.js
var add = require('./add');
var should = require('should');

describe('test add', function() {
    it('1 + 1 should be equal to 2', function(done){
        (add(1,1) === 2).should.be.true;
    });
});
```
然后保存代码之后进入终端，执行mocha add.test 即可。其中describe()函数是测试描述，表示一组相关测试用例是对哪个模块的测试。it()是一个测试用例，在一个describe块的内部可以执行对个测试用例(it块)。

这是同步函数的测试。在Node环境中，绝大部分的业务逻辑都是异步的，所以测试结果的回调是JavaScript测试框架需要解决的首要问题。Mocha提供了一个回调函数done，在业务代码执行完毕之后调用done()结束测试用例，不然的话测试用例就会出现timeout的情况导致测试用例失败。Mocha默认的超时时间是2000毫秒，如果由于测试用例的执行时间比较长需要延长超时时间，可以在命令行添加 -t 参数，比如mocha -t 3000 add.test.js。当然还有其他命令行参数，比如 mocha -w 用来监视指定的测试脚本。只要测试脚本有变化，就会自动运行Mocha。查看更多的命令行参数可以通过 mocha -h 查找。

那么接下来看一个异步测试例子：
```
describe('test async function', function(){
    it('supertest example', function(done){
        request.post('/')
        .expect(200)
        .end(function(err, res) {
            should.not.exist(err);
            done();
        })
    })
})
```
在request请求执行完毕end()之后，一定要调用done()，否则测试用例超时，那么这个用例就失败了。当然这里用到了断言库should，稍后再介绍它的语法。

### 3、钩子(hooks)
Mocha的Hooks主要是用来设置前置条件和后置处理。比如说一个业务请求之前必须先登录，那个登录操作就在before()里面完成。Mocha提供了before,after,beforeEach,afterEach四个钩子。
```
describe('hooks', function() {
  before(function() {
    // runs before all tests in this block
  });

  after(function() {
    // runs after all tests in this block
  });

  beforeEach(function() {
    // runs before each test in this block
  });

  afterEach(function() {
    // runs after each test in this block
  });

  it('test',function(done){
    //test here
  })
});
```
如果说某个测试需要跳过，使用it.skip()，Mocha跳过这个用例而不影响其他it块；只测试某个测试用例，使用it.only()，Mocha只执行.only用例，无视其他it块。

## 二、断言库should
Mocha本身是不包含断言库的，所以我们需要自己选择断言库。should是一个很简单的、贴近自然语言的断言库。当然，Mocha是适配所有的断言库的，如果你喜欢其他的断言库比如expect之类的，你也可以把它包含进来使用。

should的语法非常贴近自然语言，简单易懂，常见的should断言如下：
```
// 全等，相当于===
.exactly
(5).should.be.exactly(5)
 
// 对象存在
.ok
true.should.be.ok;
'yay'.should.be.ok;
(1).should.be.ok;
({}).should.be.ok;
false.should.not.be.ok;

// 真
.true
(5===5).should.be.true
(err === null).should.be.true;

// 相等,相当于 ==
.eql
({ foo: 'bar' }).should.eql({ foo: 'bar' });
[1,2,3].should.eql([1,2,3]);
// see next example it is correct, even if it is different types, but actual content the same
[1, 2, 3].should.eql({ '0': 1, '1': 2, '2': 3 });

// 非数字
.NaN
(undefined + 0).should.be.NaN;

// 判断类型
.typeof
user.should.be.type('object');
'test'.should.be.type('string');

// 构造函数的一个实例
.instanceof
user.should.be.an.instanceof(User);
[].should.be.an.instanceOf(Array);

// 存在
.exist()
should.not.exist(err)

//深度包含
.containDeep()
[[1],[2],[3]].should.containDeep([[3]]);
[[1],[2],[3, 4]].should.containDeep([[3]]);
[{a: 'a'}, {b: 'b', c: 'c'}].should.containDeep([{a: 'a'}]);
[{a: 'a'}, {b: 'b', c: 'c'}].should.containDeep([{b: 'b'}]);

// 抛出异常
.throw()和throwError()
(function(){
  throw new Error('fail');
}).should.throw();
(function(){
  throw new Error('fail');
}).should.throw('fail');

// http响应的头部包含
.header
res.should.have.header('content-length');
res.should.have.header('Content-Length', '123');

// 包含或等价于
.containEql
({ b: 10 }).should.containEql({ b: 10 });
([1, 2, { a: 10 }]).should.containEql({ a: 10 });
```

## 三、SuperTest
单单使用Mocha和should就几乎可以满足所有JavaScript函数的单元测试。但是对于Node应用而言，不仅仅是函数的集合，比如一个web应用的测试。这时候就需要配合一个http代理，完成Http请求和路由的测试。
Supertest是一个HTTP代理服务引擎，可以模拟一切HTTP请求行为。Supertest可以搭配任意的应用框架，从而进行应用的单元测试。

### 1、用法
#### API
首先，传入应用来实例化supertest,比如说实例化一个express：
```
var supertest = require('supertest'), 
    express = require('express');
var app = express();
var request = supertest(app);
```
接下来就可以在Mocha测试用例中直接使用request发出一个基于该web应用的http请求了。

1、.set()
如果需要设置数据，supertest的API提供了 .set 来设置，比如：
```
describe('GET /users', function(){
  it('respond with json', function(done){
    request(app)
      .get('/user')
      .set('Accept', 'application/json')
      .expect(200)
      .end(function(err, res){
        should.not.exist(err);
        res.text.should.containEql('success');
        done();
      });
  });
});
```

2、.expect()
.expect 是一个断言，上述测试代码在执行之后期望的状态码是200(OK)。如果接收到的数据为html页面，.expect('Content-Type', 'text/html;charset=utf-8')

3、.end()
.end 是执行一个request请求，在回调函数里面根据业务逻辑的返回数据做断言分析。

4、.send()
很多情况下，项目需要测试一个表单业务。supertest提供了.send()方法来发送表单域数据。比如一个登录模块，需要发送用户名密码：
```
describe('test login', function(){
    it('login sucessfully', function (done) {
        request.post('/user')
        .send({
            username: 'username',
            password: '123456'
        })
        .end(function (err, res) {
            should.not.exists(err);
            done();
        });
    });
})
```

5、.attach()
.attach方法主要用来测试文件上传，由于.send()只能上传文本域，所以关于multipart -file的上传需要通过附件来绑定。
```
request(app)
.post('/')
.field('name', 'my awesome avatar')
.attach('avatar', 'test/fixtures/homeboy.jpg')
...
```

#### 持久化Cookie
在很多业务测试中，需要用户先登录才有权限执行操作。这个时候作为HTTP请求模拟，必须要可以保存一些Cookie数据，也就是Cookie的持久化。这一般有两种解决思路。
1、在 supertest 中，可以通过 var request = supertest.agent(app) 获取一个 agent 对象，这个对象的 API 跟直接在 superagent 上调用各种方法是一样的。这个request在被多次调用 get 和 post 之后，可以一路把 cookie 都保存下来。
```
var supertest = require('supertest');
  var app = express();
  var request = supertest.agent(app);

  request.post('login').end(...);
  // then ..
  request.post('create_topic').end(...); // 此时的request中有用户登陆后的 cookie
```
2、通过.set()，在发起请求时，调用 .set('Cookie', 'a cookie string') 这样的方式。
```
var request = require('supertest');
var userCookie;
request.post('login')
.end(function (err, res) {
    userCookie = res.headers['Cookie']
});
// then ..

request.post('create_topic')
    .set('Cookie', userCookie)
    .end(...)
```

### 2、实战demo
为了更好地帮助理解SuperTest的强大功能，这里放上我的一个项目模块的测试demo。
#### 项目源文件
首先，这是一个express应用的登录注册和用户信息修改模块，要测试的就是这个user模块。为了节省篇幅，这里只放上UserController和Router两个项目文件。
```
UserController.js
// 用户首页
exports.getUser = function (req, res) {
    if (!req.user) {
        res.send({status: false, info: '未登录'});
    } else {
        res.render('user', {status: true, userdata: req.user});
    }
};

exports.login =  function (req, res, next) {
    req.session.save(function (err) {
        if (err) {
            log.error(err);
            return next(err);
        }
        res.redirect('/user/' + req.user.username);
    });
};

exports.logout = function (req, res, next) {
    req.logout();
    req.session.save(function (err) {
        if (err) {
            log.error(err);
            return next (err);
        }
        res.redirect('/user');
        res.end();
    });
};

// 用户注册
exports.addUser = function (req, res, next) {
    if (!req.body.username || !req.body.password) {
        return res.send({status: false, info: '用户名或密码不能为空'});
    } else if (req.body.password.length < 6) {
        return res.send({status: false, info: '密码长度太短'});
    } 

    User.register(new User({username: req.body.username}), req.body.password, function (err, user) {
        if (err) {
            log.error(err);
            return res.render('register', {info: '用户名已被使用'});
        } 
        passport.authenticate('local')(req, res, function () {
            req.session.save(function (err) {
                if (err) {
                    log.error(err);
                    return next(err);
                }
                res.redirect('/user/' + req.body.username);
            });
        });

    });
};

// 更新用户信息文字资料
exports.updateInfo = function (req, res, next) {
    User.findOne({username: req.user.username}, function (err, doc) {
        if (doc) {
            doc.update(req.body, function (err, data) {
                if (err) {
                    log.error(err);
                    return next(err);
                }
                res.send({status: true});
            });
        } else {
            next();
        }
    });
};

// 更换头像
exports.updateAvator = function (req, res,next) {
    upload(req, res, function (err) {
        if (err) {
            log.error(err);
            return next(err);
        }
        User.findOne({username: req.user.username}, function (err, doc) {
            if (err) {
                log.error(err);
                return next(err);
            } else if (doc) {
                doc.update({avator: req.file.filename}, function (err, data) {
                    if (err) {
                        log.error(err);
                        return next(err);
                    }
                    res.type('html');
                    res.send({status: true, avator: req.file.filename});
                });
            } else {
                return next();
            }
        });
    });
};
```
Router文件
```
router.get('/:uid', isAuthenticated, user.getUser);
router.get('/', function (req, res) {
    res.render('login', {info: ''});
});
router.post('/',passport.authenticate('local'), /*{failureRedirect: '/user'}),*/ user.login);

router.post('/authenticate/logout', user.logout);

router.get('/authenticate/register', function (req, res) {
    res.render('register', {info: ''});
});

router.post('/authenticate/register', user.addUser);

// 用户相关的资料
router.post('/update/info', user.updateInfo);
router.post('/update/avator', user.updateAvator);

module.exports = router;
```

#### 测试脚本
首先，测试注册功能。为了防止用户名冲突，在测试用例之前设置钩子before块，清除User集合。
```
 // user.test.js
 describe('test user.addUser()', function () {
        before(function (done) {
            User.remove({}, function (err) {
                done();
            });
        });

        it('register sucessfully', function (done) {
            request.post('/user/authenticate/register')
            .send({
                username: 'username',
                password: '123456'
            })
            //注册成功之后重定向
            .expect(302, function (err, res) {
                should.not.exists(err);
                done();
            });
        });
        //测试用户名存在的用例
        it('username already exist', function (done) {
            request.post('/user/authenticate/register')
            .send({
                username: 'username',
                password: '123456'
            })
            .end(function (err, res) {
                should.not.exists(err);
                res.text.should.containEql('用户名已被使用');
                done();
            });
        });
    });
    // 测试头像上传
    describe('test user.updateAvator()', function () {
        //bofore HOOKS，测试业务逻辑之前先登录，并持久化cookie
        before(function (done) {
            request.post('/user')
            .send({
                username: 'username',
                password: '123456'
            })
            .end(function (err, res) {
                done();
            });
        });

        it('update avator upload sucessfully', function (done) {
            request.post('/user/update/avator')
            .attach('avator','test/image/test.jpg')
            .end(function (err, res) {
                should.not.exists(err);
                res.text.should.containEql('"status":true');
                done();
            });
        }); 
    });
```

