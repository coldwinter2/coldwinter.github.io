---
title: Express + socket.io 聊天程序
date: 2016-11-17 13:44:59
tags: 
- webSocket
- socket.io
category: 
- node.js
- webSocket
---


### 基础环境


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


### 详细说明

#### 服务端

先帖代码

```javascript
var express = require('express');
var app = express();
server = app.listen(8000,function()
{
    console.log('服务器启动:http://localhost:8000');
});
io = require('socket.io').listen(server);
	
app.use('/',express.static(__dirname + '/www'));
	
//在线用户
var onlineUsers = {};
//当前在线人数
var onlineCount = 0;
	
io.on('connection', function(socket){
	
	console.log(socket.id);
	if(!onlineUsers.hasOwnProperty(socket.id)) {
		onlineUsers[socket.id] = socket;
		//在线人数+1
		onlineCount++;
	}
	
	socket.emit('login',{user:socket.id, count:onlineCount});
	
	io.emit('enter', {count:onlineCount, user:socket.id});
	console.log('用户'+socket.id + " 加入的房间，当前在线人数:" + onlineCount);
	//监听用户退出
	socket.on('disconnect', function()
	{
		//将退出的用户从在线列表中删除
		if(onlineUsers.hasOwnProperty(socket.id)) {
			//退出用户的信息
			//删除
			delete onlineUsers[socket.id];
			//在线人数-1
			onlineCount--;
	
			//向所有客户端广播用户退出
			io.emit('logout', { count:onlineCount, user:socket.id});
			// console.log(socket.id+'退出了聊天室');
		}
        console.log(socket.id +'退出了房间,当前房间人数:' + onlineCount);
	});
	
	//监听用户发布聊天内容
	socket.on('msg', function(obj){
		//向所有客户端广播发布的消息
		io.emit('msg', {user:socket.id, msg:obj,count:onlineCount});
		console.log(socket.id+'说：'+obj);
	});
});
```


##### 说明：

不同于网上其它的例子，这里直接用express的对象启动监听，就可以返回一个server对象。
如果希望监听在逻辑的最后执行，可以先执行创建server(`createServer`)，再监听。
另外，选择8000端口，是因为，在mac下，访问80端口需要相应的权限，因此避免用此端口可以免去开发中的一些麻烦。等正式环境再做调整。



#### 客户端

客户端的代码结合的jquery来使用，我只帖重点部分。


```javascript
<body>
    <script src="socket.io/socket.io.js"></script>
    <script src="js/jquery-3.1.1.min.js"></script>
    <script>


    var user_id = '';


    $(document).ready(function(){

        var socket=io.connect();//与服务器进行连接

        $("#send").click(function(){
            var data = $("#text").val();
            socket.emit('msg', data );
            $("#msg").val( $("#msg").val() + "\n我:" + data ); 
        })

        socket.on('enter',function(data){
            
            $("#online").val('当前在线人数:' + data.count);

        });

        socket.on('logout',function(data){
            $("#online").val('当前在线人数:' + data.count);
        });

        socket.on('login',function(data){
            user_id = data.user;
            $("#online").val('当前在线人数:' + data.count);
        });

        socket.on('msg',function(data){
            $("#online").val('当前在线人数:' + data.count);
            if(data.user == user_id) return;
            $("#msg").val( $("#msg").val() + "\n" + data.user +':'+data.msg);


        })

    })

    </script>
    ......
```
   
##### 说明
引入的js文件 `socket.io/socket.io.js` 这个是由服务器创建的访问，我们不必真的创建一个目录来存放一个真实的socket.io.js文件，当然，如果你的站点动静分离，或者有特别的需求，可以从网站下载相应的文件，放在相应的目录下提供访问，也是完全没有问题的。

具体socket的绑定事件，可以从[socket.io官方](http://socket.io)的文档里找到相应的内容。


### 写在最后

本文只是为了贴一段代码，做一个备忘。这部分内容还会继续深入学习，这个笔记很持续更新一段时间，直到我搞清楚webSocket为止。

下面附上我自己的各组件版本

```
node.js    v6.9.1
socket.io  1.5.1
socket.io_client  1.4.5
express    4.14.0

```

踩过此坑，特此记录