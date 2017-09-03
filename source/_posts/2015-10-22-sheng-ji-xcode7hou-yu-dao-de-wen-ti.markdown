---
layout: post
title: "升级xcode7后遇到的问题"
date: 2015-10-22 14:08:21 +0800
comments: true
categories: iOS
---
##前言
升级xocde7后遇到一些问题，先记录如下

###xcode7网络请求报错
错误如下:

The resource could not be loaded because the App Transport Security policy requires the use of a secure connection.}
<!-- more -->
后来发现苹果最新的xcode建议使用https协议，
![https](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151022-0.png)

如果不想使用https协议，可以通过修改info.plist达成目的，
在`Info.plist`中添加`NSAppTransportSecurity`类型`Dictionary`。
在`NSAppTransportSecurity`下添加`NSAllowsArbitraryLoads`类型`Boolean`,值设为`YES`


因为我的工程做了国际化了，所在再两个info.plist中都加了如上的配置，但是依然报错，后来在工程的target的info中修改。OK
截图如下![demo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151022-1.png)


###URL scheme
URL scheme一般使用的场景是应用程序有分享或跳其他平台授权的功能，分享或授权后再跳回来。
之前的ios版本都没有问题，但是在ios9做了限制,不配置的话会报错如下:
![demo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151027-0.png)

配置方案：是要在info.plist中设置 LSApplicationQueriesSchemes 类型为数组
![demo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151027-1.png)

###Bitcode

   
Bitcode is an intermediate representation of a              compiled program. Apps you upload to iTunes Connect that contain bitcode will be compiled and linked on the App Store. Including bitcode will allow Apple to re-optimize your app binary in the future without the need to submit a new version of your app to the store.
       
       
 
 
 bitcode是被编译程序的一种中间形式的代码.包含bitcode配置的程序将会在App store上被编译和链接。bitcode允许苹果在后期重新优化我们程序的二进制文件，而不需要我们重新提交一个新的版本到App store上.
 
 bitcode也允许苹果在后期重新优化我们程序的二进制文件，有类似于App瘦身的思想
 
用xcode7报错如下:
	
	XXXX’ does not contain bitcode. You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE), obtain an updated library from the vendor, or disable bitcode for this target. for architecture arm64
问题的原因是：某些第三方库还不支持bitcode。要不然是等待库的开发者升级了此项功能我们更新库，要不就是把这个bitcode禁用

禁用的方法就是找到如下配置，设置为NO
![demo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151027-2.png)

 
 
