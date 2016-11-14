---
title: Express + socket.io 聊天程序踩坑实录
date: 2016-11-14 13:20:13
tags: 
- webSocket
- socket.io
category: 
- node.js
- webSocket
---

### 起因

&nbsp;&nbsp;&nbsp;&nbsp;本文其实没有其实没有很特别的内容，只不过是写一个简单的node.js下express+socket.io的基础示例。不过就是这样一个网上随处都可以找到的示例，却让我踩了不小的一个坑。含坑的教程有很多，我随便贴一些连接上来，基本错的都一样，大部分都是复制粘贴来的吧。例如：


[使用Node.js+Socket.IO搭建WebSocket实时应用](http://www.open-open.com/lib/view/open1402479198587.html)

[Node.js + Web Socket 打造即时聊天程序嗨聊](http://www.cnblogs.com/Wayou/p/hichat_built_with_nodejs_socket.html)

[NodeJS+Express+Socket.io的一个简单例子](http://www.tuicool.com/articles/fmeQVjZ)


下面，我就以一个聊天程序来说明一下踩坑的过程。


1. 下载安装node.js(自行百度）
2. 安装npm（自行百度）
3. 创建工程目录 ` mkdir webSocket ` 并且切换到该目录
4. 在webSocket下创建npm工程 ` npm init ` 按照提示完成
5. 安装express ` npm install express ` (注意，此处非全局安装)
6. 安装socket.io ` npm install socket.io ` (同上)
7. 创建web目录www ` mkdir www `
8. 创建js目录 ` mkdir www/js ` 
9. 在js目录中安装socket.io.js,这个文件在 `node\_modules/socket.io\_client` 内
10. 创建相应的 `index.js`、`www/index.html`文件

OK，基本的环境搭建到此结束。下面开始踩坑。


### 踩坑过程

##### 1.Client端引用Socket.io.js的问题

首先说，socket.io不是一定要放在socket.io文件夹内的，几乎网上大部分教程，代码里都写着

```html
<script src="socket.io/socket.io.js"></script>
```

其实这个socket.io.js可以放在任何位置，然而很多人发现了，按照教程，socket.io怎么也无法连接成功。而且打开浏览器的web选项，还能看到一个http请求 http://localhost:xxxx/socket.io/?EIO=xxxx  返回了404错误。很可能会联想到，和这个文件的路径有关。
初学者很容易联想到奇奇怪怪的规则，其实和这个没关系，而和服务端的代码有关。
这就引出了教程中的第二个坑。


##### 2.服务端代码的坑

和前面一个坑相比，这个才是正经的坑，上面那个只可能影响到一些喜欢深入思考的初学者，但是下面这个坑，绝对会影响到所有初学者。


含坑代码

```javascript
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.get('/', function(req, res){
  res.sendfile('index.html');
});

io.on('connection', function(socket){
  console.log('a user connected');
    
  socket.on('chat message', function(msg){
    console.log('message: ' + msg);

    io.emit('chat message', msg);
  });

});

app.set('port', process.env.PORT || 3000);

var server = http.listen(app.get('port'), function() {
  console.log('start at port:' + server.address().port);
});
```

我随便从前面的教程连接里摘出来的一段主要代码。
这段代码的坑在于，express是一个完整的http服务器框架，因此不再需要http这个库了。
在建立本地监听的过程中，不需要 `require('http').Server(app)` 这种写法了，直接调用express的listen方法就可以了，例如：

```javascript
var express = require('express');
var app = express();
server = app.listen(8000,function()
{
    console.log('服务器启动');
});
io = require('socket.io').listen(server);

// some other code

```

如果使用http包来创建，则对应的socket.io的相关功能就没有了。

### 踩坑分析

其实这个坑，并不是原博文作者的问题，而是版本迭代问题。
大概是express经过几个版本的迭代在创建上有一些变化。但是远博文没有说明`node.js` `socket.io`和`express`的版本。

所以我一定要附上我所用环境的版本。

```
node.js    v6.9.1
socket.io  1.5.1
socket.io_client  1.4.5
express    4.14.0

```

踩过此坑，特此记录