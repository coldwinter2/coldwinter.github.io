---
title: 在cocos2dx-lua中调试lua代码
date: 2017-03-08 11:09:29
tags: cocos2dx lua debug vscode
category: cocos2dx
---



lua是游戏开发中常用的脚本语言。本文主要讲述如何在vscode中搭建针对cocos2dx-lua的debug环境，以实现断点和单步调试。

## 基础环境

首先所需的基础环境包括:
vscode 
node.js
cocos2dx-lua(我使用的是3.14.1版本，其他版本尚未测试,理论上2.x版本也可用)


## lua调试环境搭建

#### Step 1: 安装vscode扩展插件 `lua Debugger` 

![图1](/img/cocos_lua_debug/lua_0.png)

之后按照配置说明，下载`vscode-debuggee.lua`并且复制到cocos2dx-lua的src目录下（目录随意，只要require的时候能够require到就可以），这里需要注意一下引入debuggee的代码，第一行的`local json = require 'dkjson'`不是必须的，这里只需要提供一个，包含 encode和decode方法的json包即可，可以用cjson也可以用cocos2dx-lua自带的json.lua，这个json工具是用来解析和发送调试指令的。


#### Step 2: 配置Debug插件

在debug面板增加调试配置，选择`Lua Debugger`，如图
![](/img/cocos_lua_debug/lua_1.png)

之后会打开配置选项
![](/img/cocos_lua_debug/lua_2.png)
我们用到的设置，只有最后一个`wait`的配置。这里需要重点关注`sourceBasePath`用来设置代码的根目录，当断点触发时，调试器会根据相对路径拼接上这个基础路径，来查找对应的lua代码，如果不设置正确，那么调试的时候将找不到对应的lua代码，导致调试器报错。

#### Step 3: 引入调试插件

设置好后，我们去修改cocos2dx-lua的代码，
![](/img/cocos_lua_debug/lua_3.png)

打开cocos2dx-lua的默认入口lua文件`controller.lua`，在这个文件的合适位置（合适位置的意思是，保证json是可用的，如果引入其他的json库，则不必将代码放在上图所示位置)加入启动调试器的代码。如果不是本机调试，还需要设置IP地址和相应的端口号。具体请参考`Lua Debugger`的配置帮助。

#### Step 4： 开启调试之旅

最后，随便找个文件，打一个断点（文中断点位置为AccelerometerTest.lua），然后启动调试器，会有下图所示提示。


![](/img/cocos_lua_debug/lua_4.png)


最后打开你的vs/xcode/android/ios 工程，运行（我这里是在windows下用vs运行了win32项目工程),启动后，等待连接界面消失，表示调试器已经成功连接。
然后触发断点，断点成功，如图所示：

![](/img/cocos_lua_debug/lua_5.png)

左侧可以看到变量内容，右侧鼠标指向相应的变量，可以看到具体的变量信息。可以单步逐行执行代码。

至此，cocos2dx-lua的调试环境搭建完成。




## 写在最后

vscode还是十分强大的，cocos2dx-js的调试环境搭建相对更加简单一些,后续会整理一篇搭建cocos2dx-js调试环境的文章。







