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

&nbsp;&nbsp;&nbsp;&nbsp;本文其实没有其实没有很特别的内容，只不过是写一个简单的node.js下express+socket.io的基础示例。不过就是这样一个网上随处都可以找到的示例，却让我踩出了一个不大不小的坑。教程有很多，我随便贴一些连接上来，基本写法都一样，大部分应当都是复制粘贴来的。例如：


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
8. 创建js目录 ` mkdir www/js ` （这一步在本文内无用，是用来放客户端js代码的)
9. 在js目录中安装socket.io.js,这个文件在 `node\_modules/socket.io\_client` 内
10. 创建相应的 `index.js`、`www/index.html`文件

OK，基本的环境搭建到此结束。下面开始踩坑。


### 踩坑过程

#### 1.Client端引用Socket.io.js的问题

首先说，socket.io不是一定要放在socket.io文件夹内的，几乎网上大部分教程，代码里都写着

```html
<script src="socket.io/socket.io.js"></script>
```

这让很多刚来入门的新人，以为，需要吧socket.io.js放在一个叫做socket.io的文件夹里，并且提供访问。其实是没必要的。
因为已经存在一个叫socket.io\_client的包，在socket.io在创建的时候会自己注册一个socket.io的url访问，因此用户无需干预。

其实这个socket.io.js也可以放在任何位置，只要从[socket.io](http://socket.io/)下载对应的socket的独立js文件，就当做普通的js一样一用就可以了，无需了解服务端内部究竟干了什么。

然而很多人发现了，按照教程，socket.io怎么也无法连接成功。如果自己手动引入了别的socket.io.js而且打开浏览器的开发者工具，就会看到一个http请求 http://localhost:xxxx/socket.io/?EIO=xxxx  返回了404错误。由此很可能会联想到，和这个文件的路径有关。

其实这个错误和这个没关系，而和服务端的代码有关。
这就引出了教程中的第二个坑。


#### 2.服务端代码的坑

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

要避免坑1，就要去阅读socket.io的api文档，里面写的很清楚，如果创建socket.io的时候没有带路径参数，默认创建在socket.io路径下，可以通过如下代码：

```javascript
io = require('socket.io')(server,'js');
```

这样就可以把 `<script src="socket.io/socket.io.js"></script>` 换成 `<script src="js/socket.io.js"></script>` 了


第二个坑，大概是express经过几个版本的迭代在创建上有一些变化。因为原博文没有说明`node.js` `socket.io`和`express`的版本。

所以写教程第一步就是说明当前环境，如果后续有人发现代码无法测试通过，可以先去找对应的版本试试

下面附上我自己的测试环境

```
node.js    v6.9.1
socket.io  1.5.1
socket.io_client  1.4.5
express    4.14.0

```

踩过此坑，特此记录