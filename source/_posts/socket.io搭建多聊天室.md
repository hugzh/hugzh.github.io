title: socket.io搭建多聊天室
date: 2016-01-05 22:41:23
tags:
    - node
    - 网络
categories:
    - Nodejs
---

html5新兴的API给前端开发者带来了难以抑制的兴奋，除了前端的API，html5还提供了一些有里程碑意义的前后端通信API，WebSocket就是这其中之一。WebSocket的目标是在一个单独的持久连接上提供全双工、双向通信。
<!--more-->
## WebSocket前世今生
WebSocket是html5的一种新协议，在它出现之前，浏览器与服务器之间通过http只能实现单向的通信。在此之前，要实现浏览器之间的即时通信，一般采用Comet来模拟。Comet是一种服务器向页面推送消息的技术，这种技术可以让信息实时推送到页面，也算是可以比较完整地实现实时通信，但是效率低下。
实现Comet的方式主要有两种，一种是轮询(polling)，一种是流。轮询，原理简单易懂，就是客户端通过一定的时间间隔以频繁请求的方式向服务器发送请求，来保持客户端和服务器端的数据同步。问题很明显，当客户端以固定频率向服务器端发送请求时，服务器端的数据可能并没有更新，带来很多无谓请求，浪费带宽，效率低下。第二种实现Comet的是HTTP流。流不同于轮询，它在页面的整个生命周期内只使用一个HTTP连接，通过服务器端语言的缓冲区刷新机制来将消息推送出去。
除了Comet，通过Flash的API也是可以实现Socket的。AdobeFlash 通过自己的 Socket 实现完成数据交换，再利用 Flash 暴露出相应的接口为 JavaScript 调用，从而达到实时传输目的。此方式比轮询要高效，但是随着今年Flash正式宣告退出历史舞台，这种方式现在的存在意义不大。
从上文可以看出，传统 Web 模式在处理高并发及实时性需求的时候，会遇到难以逾越的瓶颈，我们需要一种高效节能的双向通信机制来保证数据的实时传输。在此背景下，基于 HTML5 规范的、有 Web TCP 之称的 WebSocket 应运而生。
![暴走](http://wanzao2.b0.upaiyun.com/system/pictures/11/original/28.png)
## WebSocket的原理
WebSocket 是一种双向通信协议，在建立连接后，WebSocket服务器和 Browser/Client Agent 都能主动的向对方发送或接收数据，就像 Socket一样。它类似TCP长连接，但是WebSocket使用的是ws和wss两种通信协议，所以这是完全不同于HTTP的一种网络协议，尽管它也是默认使用80和443端口。
WebSocket的连接包括第一步握手和第二步数据交换。在通过WebSocket开始双向通信时，首先需要与服务器建立连接。而用于建立连接的请求是由客户端发起，服务器端将会确认连接对象的源以及协议，并发送连接许可的响应。在发送了响应之后，浏览器会将该连接升级为WebSocket。握手成功之后就建立起长连接，直到服务器或者浏览器某一方主动断开连接。客户端发起连接非常简单，只需要一句Javascript即可实现：
```
var ws = new WebSocket('ws://www.example.com/bar')
```
然后就是第二步，浏览器和服务器之间双向数据发送了。Nodejs是即时通信应用很完美的开发语言（此处不是为了引起撕逼），很自然地，Nodejs本身也比较完善地实现了WebSocket相关API。客户端（浏览器）发送消息到服务器最简单的方法就是通过调用socket对象的send()方法，然后Node服务器通过注册message事件进行监听，当然服务器send过来的消息也是通过对客户端message事件进行捕捉，从而实现双向的数据交互。这两者之间交换的数据可以是字符串、ArrayBuffer（二进制数据）或者Blob，所以WebSocket也是可以传输文件的。也可以通过广播事件将消息发送到所有连接到服务器的客户端，也就是这篇文章要介绍的聊天室搭建的技术基础啦(尼玛终于到题目了)。
![鸡冻](http://wanzao2.b0.upaiyun.com/system/pictures/19/original/45.png)

## Socket.io开发聊天室
Socket.io是基于Nodejs生态的，但是做Nodejs所不能实现的WebSocket的库。就像Express之于Nodejs、jQuery之于Javascript，Socket.io是对Node语言关于WebSocket所有API的封装和拓展，是一个应用框架。考虑到不同浏览器对WebSocket的支持程度不同，Socket.io通过支持多种协议（比如前面说的轮询）来实现良好的兼容，它会自动根据浏览器选择适合的通讯方式，从而让开发者可以聚焦到功能的实现而不是平台的兼容性。Socket.io[官方网站](http://socket.io/)里面也介绍了Socket.io的主要用途——包括实时数据分析应用、在线聊天、文档合作等等。这里主要是简单介绍一下如何使用Socket.io+Express去构建一个简单的在线聊天室。
两个重要需求点：
* 1、多个聊天室，根据url路径划分当前聊天室
* 2、文本聊天

### 第一步：搭建Express服务器
首先生成Express应用，进入根目录下的server.js文件编辑服务器端。由于Express和socket.io都是安装了最新版本（分别是4.x和1.3.7），所以一些配置上面和Express 3.X还是不一样的，主要是注意一下配置server监听的时候要先创建一个由express对象（app）实例化的httpServer（server）对象，然后引入socket.io对这个对象进行监听，如下：
```
var PORT = 80;
...
var app = express();
var server = http.Server(app);
var io = require('socket.io').listen(server);
...
```
最后server.js的端口监听要绑定在这个httpServer上面：
```
...
if (!module.parent) {
    // This server is socket server
    server.listen(PORT);
    log.info('Weiku started up');
}
···
```

### 第二步：实现服务器端的逻辑
搭建完Express之后，就需要实现服务器的事件监听和响应了。继续对上面的server.js进行编辑。后台的所有事件注册应该建立在io对象的connection事件之上，也就是说需要在浏览器端已经开启，请求向服务器连接的基础上进行。最基本的事件包括消息事件message、断开事件disconnect，连接事件connection。由于这里需要根据路径划分聊天房间，需要进行命名空间划分。Socket.io提供.join(roomid)方法加入聊天室，.leave(roomid)方法离开聊天室，.to(roomid).emit(event)方法向roomid的所有连接用户的群发消息。首先需要划分路由，从而区分聊天室号：
```
io.on('connection', function (socket) {
    // 获取用户当前的url，从而截取出房间id
    var url = socket.request.headers.referer;
    var split_arr = url.split('/');
    var roomid = split_arr[split_arr.length-1] || 'index';
    ...
});
```
然后是对join、message和disconnect三个事件的监听：
```
...
    var user = '';
    socket.on('join', function (username) {
         user = username;
        // 将用户归类到房间
        if (!roomUser[roomid]) {
            roomUser[roomid] = [];
        }
        roomUser[roomid].push(user);
        socket.join(roomid);
        socket.to(roomid).emit('sys', user + '加入了房间');
        socket.emit('sys',user + '加入了房间');
    });

    // 监听来自客户端的消息
    socket.on('message', function (msg) {
        // 验证如果用户不在房间内则不给发送
        if (roomUser[roomid].indexOf(user)< 0) {  
          return false;
        }
        socket.to(roomid).emit('new message', msg,user);
        socket.emit('new message', msg,user);
    });

    // 关闭
    socket.on('disconnect', function () {
        // 从房间名单中移除
        socket.leave(roomid, function (err) {
            if (err) {
                log.error(err);
            } else {
                var index = roomUser[roomid].indexOf(user);
                if (index !== -1) {
                    roomUser[roomid].splice(index, 1);
                    socket.to(roomid).emit('sys',user+'退出了房间');
                } 
            }
        });
    });
```
### 第三步：实现客户端交互
客户端的消息收发主要是监听来自服务器emit的事件及其消息内容。由于socket.io通信事件是可以自定义的，所以前后端的事件监听必须对应好。为了方便，这里浏览器端的界面比较简单：一个消息展示窗口和一个文本输入框。当用户在输入框输入文本并敲回车键的时候，客户端调用socket.send()方法向服务器发送消息。浏览器接收到来自服务器的sys事件和new message事件之后，将消息内容显示在消息展示窗口。
index.ejs:
```
    <div>我的名字:<span class="name"></span></div>
    <div id="chat-area" style="width:300px;height:400px;background-color:#eef;overflow-y:scroll">
        <div class="messages">
        </div>
    </div>
    <input type="text" class="inputMessage" placeholder="按回车键发送" />
   <script src="/socket.io/socket.io.js"></script>
   <script src="/js/jquery.min.js"></script>
   <script src="/js/chat.js"></script>
```
index界面主要引入三个js文件，其中socket.io.js的路径大家可能觉得不可思议，明明项目中并没有“/socket.io”这个路径。这是由于socket.io安装完毕之后自动完成了这个路径的映射，所以我们是不用管这个的，照写就可以了。chat.js主要负责客户端和服务器的事件通信,它通过 var socket = io() 创建浏览器端的socket.io对象。
chat.js:
```
$(function () {
    var username =  prompt('请输入昵称');
    $('.name').html(username)
    var input = $('.inputMessage');
    // 默认链接到渲染页面的服务器
    var socket = io();
    function scrollToBottom () {
        $('#chat-area').scrollTop($('#chat-area')[0].scrollHeight);
    };

    socket.on('connect', function () {
        var name = $('.name').text() ||'匿名';
        socket.emit('join',name);
    })
    socket.on('sys', function (msg) {
        $('.messages').append('<p>'+msg+'</p>');
        // 滚动条滚动到底部
        scrollToBottom();
    });
    socket.on('new message', function (msg,user) {
        $('.messages').append('<p>'+user+'说：'+msg+'</p>');
        // 滚动条滚动到底部
        scrollToBottom();
    });
    input.on('keydown',function (e) {
        if (e.which === 13) {
            //判断回车键
            var message = $(this).val();
            if (!message) {
                return ;
            }
            socket.send(message);
            $(this).val('');
        }
    });
});
```
到这里，一个简单的多聊天室应用就完成了，由于在Express的Router设置了访问路径:
```
router.get('/room/:id', function(req, res, next) {
  res.render('index');
});
```
安装完毕应用，执行server.js开启服务器之后，打开浏览器访问 localhost:8080/room/1 就可以访问房间号为1的聊天室了。然后呢，鉴于...
![demo](http://i4.tietuku.com/d121eb715f0f6654s.png)
业界良心，还是要附上全部的[demo](https://github.com/hugzh/chatroom-demo)代码的。

当然，你还可以根据自己的情况把demo拓展得更完善一些，比如加点好看的css，实现用户登录等等。如果需要实现登录功能，服务器socket应该添加对session读写的支持。由于socket.io本身是不支持session的，需要引入第三方模块比如 socket.io-express-session ，来实现用户状态认证：
```
if (socket.handshake.session.passport) {    
    user.username = socket.handshake.session.passport.user;
}
```

## 结尾
到此，一个简单的socket.io应用就完成了，这篇文章主要是解决Express4.x和socket.io1.3.x配置的时候需要调整的一些写法，不注意的话很容易各种瞎折腾。另外，本文demo部分参考了吴彦欣老师的一个[demo](https://github.com/wuyanxin/chatapp-demo)，感谢分享。


