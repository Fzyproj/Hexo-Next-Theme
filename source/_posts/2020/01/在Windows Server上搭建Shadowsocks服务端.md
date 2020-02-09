---
title: 在Windows Server上搭建Shadowsocks服务端
date: 2020-01-21 13:38:50
tags: 
	- 网络技术
categories: 技术类
---
想要跨越那道鸿沟却无可奈何？
如何在windows server上搭建一台属于自己的ss服务端，
下面我们一起来看...
<!--more-->
## 下载 libQtShadowsocks
github传送门：[libQtShadowsocks](https://github.com/shadowsocks/libQtShadowsocks/releases) 下载适合自己的版本，一般最新版就可以
## 配置config和bat文件
新建一个名为 libQtShadowsocks 的文件夹，将下载好的 shadowsocks-libqss-v1.8.4-win64.7z 解压进文件夹中，在文件夹中新建名为 config.json 的配置文件，内容如下
```bash
{
    "server":"0.0.0.0",
    "server_port":8898,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"Your Password",
    "timeout":600,
    "method":"aes-256-cfb",
    "http_proxy": false,
    "auth": false
}
```
在文件夹中新建名为 shadowsocks-server.bat 的批处理文件，内容如下:
```bash
@echo off
shadowsocks-libqss.exe -c config.json -S
```
然后运行 shadowsocks-server.bat 即可，关闭时就关闭批处理就行了，很简单。

## 隐藏bat窗口
运行后上面那个黑色的窗口一直存在，太影响正常使用了，下面通过vbs脚本的方法将批处理脚本隐藏
新建一个shadowsocks.vbs文件写入下面代码。
```bash
Set ws = CreateObject("Wscript.Shell")  
ws.run "cmd /c  shadowsocks-server.bat",vbhide
```