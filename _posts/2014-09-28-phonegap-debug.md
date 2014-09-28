---
layout: post
title:  "phonegap调试总结"
date:   2014-09-29 12:23
categories: phonegap 移动web调试
tag: phonegap
---

## 写在前面

phonegap作为一个支持快速开发hybrid app的开发平台，同开发web app一样需要在无线环境下调试。但是不同的是，我们写的代码是运行在phonegap里的，给调试带来了一些难度，特别是当我们需要调用设备的原生功能时，例如定位，摄像头等。下面就自己在学习过程中对phonegap调试做个总结。

## PhoneGap Developer App

从phonegap3.0开始，官方推出了 [PhoneGap Developer App](http://app.phonegap.com/) ，主要作用是在PC下运行一个命令就自动建立起一个http服务，在手机端装上mobile app后连接上PC端开启的http服务则可以让我们的代码实时反应在手机端。这样调试起来就方便了许多，不用再编译到手机上才能观察了。但是这样存在的问题是，当我们想要类似打开chrome developer tool那样调试页面的DOM元素样式时怎么办呢？

## WEINRE

全称：web inspector remote，是目前最常用的远程web调试器之一。它可以让我们在PC端像在chrome developer tool下修改代码，实时在手机端观察。除了可以实时修改DOM样式外，还可以在控制台查看js对象等。

优点：可在PC端修改css样式，实时在手机端查看；可以查看网络请求；可在控制台访问js对象
缺点：无法做网络资源代理，当需要对网络请求断点，修改请求或响应时无法满足需求；不能断点调试js

具体的安装和使用这里就不介绍了，请自行google。

## Charles/Fiddler

无线开发调试网络请求是最刚需，特别是需要模拟网速，断点请求，修改响应等时显得非常重要。phonegap开发更是如此，因为它不能像在PC端那样利用简便的chrome开发人员工具来调试。如果是在mac下开发强烈推荐Charles，功能齐全使用也简单，window下无疑是fiddler了。

优点：使用Charles可以方便地模拟移动网速，代理本地文件，修改请求响应等，是移动开发网络调试的不二选择。

## GapDebug

上面提到的还有一个很关键的问题没有解决：js断点调试，这个一直是无线端开发比较难搞的一点。对于普通的web app，例如在ios Safari下可开发调试模式在PC端的Safari下断点调试，还是很方便的。但是对于那些嵌入在web view里面的web就显得很无力了。

GapDebug是专门针对phonegap或Cordova app的一款真机调试工具，也是刚推出不久的一款真机调试神器。

优点：可断点调试js，真机调试；支持同时打开多个app调试；该有的web调试功能它都几乎具备了
缺点： 必须先编译安装到设备上才能连接到PC进行调试，所以如果不是非常必要还是不要用这种方式来调试了，很耗时间。

## 总结

总结起来，phonegap开发调试的步骤大概如下：

1. 通过`console.log & alert`来初步调试
2. 如果不涉及调用设备原生api，先注释掉引用的cordova.js并从项目目录创建软连接到本地服务器根目录在本地服务器上浏览调试
3. 需要在真机上观察调试时，使用weinre来真机调试css样式
4. 需要调试网络请求时，打开charles来调试网络请求
5. 调用到了设备原生API，并迫不得已时先编译安装app到手机上再使用gapdebug来真机断点调试

以上就是phonegap开发调试的大概步骤，可以开始布置工作区的时候就一并把这些工具都开启了，需要时再查看使用。
