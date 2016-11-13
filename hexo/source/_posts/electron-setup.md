---
title: Electron 安装
date: 2016-11-12 12:12:45
tags: Electron
category: Electron
---

&nbsp;&nbsp;&nbsp;&nbsp;对于node.js的各种模块安装，最头疼的莫过于下载了。安装包体积巨大，GFW功能强大，服务器在国外，下载成功的几率很小。因此Electron的安装也是很麻烦的。

### 解决方案：
1、在安装步骤执行到 ` > node install.js ` 时，按ctrl+c中断过程
2、阅读install.js的源代码
3、找到使用本地缓存的逻辑
4、手动下载zip文件，建立本地缓存，之后使用本地文件进行安装


### 步骤：
1、下载所需的zip包，我在mac环境下的下载路径是
> [https://github.com/electron/electron/releases/download/v1.4.5/electron-v1.4.5-darwin-x64.zip](https://github.com/electron/electron/releases/download/v1.4.5/electron-v1.4.5-darwin-x64.zip)

2、本地缓存目录为
> ~/.electron/electron-v1.4.5-darwin-x64.zip

3、手动下载的zip包放到缓存目录所在位置。

4、重新执行 ` npm install -g electron ` 即可


### 授人以渔
上面的下载地址和缓存目录是通过阅读install.js脚本，尝试执行并打印部分变量得到的。
对于很多其它的使用npm安装的功能模块，都可以通过这个方法，解决下载问题。