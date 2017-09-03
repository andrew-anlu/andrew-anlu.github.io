---
layout: post
title: "微信登录"
date: 2015-10-12 10:44:41 +0800
comments: true
categories: iOS
---
微信登录是一个很常见的功能，现在很多应用都集成了微信登录。

##准备工作
刚下载下来微信的官方demo后，一直运行报错，后来用真机运行demo，可以成功，然后把相关的包拷贝到应用项目中，一直报错，如图
![编译报错](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151012-0.png)

这个错误是没有导入库 libc++.dylib 导致，General -> Linked Frameworks and Libraries 中添加 libc++.dylib 文件即可

##第一步
