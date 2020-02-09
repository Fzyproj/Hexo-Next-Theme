---
title: Win 10 搜索框输入内容无法检索解决
tags:
  - 搜索空白
  - 小娜
date: 2020-02-07 13:04:37
categories: Windows
---

​	

今天在使用电脑的时候发现左下角开始栏不能搜索东西了，每次输入首字母想检索的时候，都会弹出一片空白让人头疼，今天我们就来解决一下这个问题。。

<!--more-->

## 打开powershell(管理员身份）

1.1 电脑输入win+x，选择powershell管理员模式点击。

## 输入代码

```
Get-AppXPackage -Name Microsoft.Windows.Cortana | Foreach {Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"}
```

即可解决上述问题。