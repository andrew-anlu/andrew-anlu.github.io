---
layout: post
title: "升级Xcode8后注意的问题"
date: 2016-10-09 16:37:48 +0800
comments: true
categories: 
---

现在已经可以从AppStore升级到Xcode8的正式版了，但是升级之后会有一些莫名其妙的问题，再次总结如下：

## 杂乱无章的Bug日志

<!--more-->

更新xcode8后，新建立工程，都会打印一堆莫名其妙看不懂的log,比如：
![1](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20161009-0.png)

屏蔽的方法如下:

xcode8里边的 `Edit Scheme->Run->Arguments,`在Environment Variables里边添加 `OS_ACTIVITY_MODE ＝ Disable`

![1](http://7xkxhx.com1.z0.glb.clouddn.com/707724-e81adf182229475f.png)



## 权限及相关设置
我们需要打开info.plist文件添加相应权限的说明，否则程序在iOS10上会出现崩溃，具体如下图

![1](http://7xkxhx.com1.z0.glb.clouddn.com/707724-d118ca12029c78ab.png)

* 麦克风权限 Privacy - Microphone Usage Description 是否允许此App使用你的麦克风？
* 相机权限： Privacy - Camera Usage Description 是否允许此App使用你的相机？
* 相册权限： Privacy - Photo Library Usage Description 是否允许此App访问你的媒体资料库？
* 通讯录权限： Privacy - Contacts Usage Description 是否允许此App访问你的通讯录？
* 蓝牙权限：Privacy - Bluetooth Peripheral Usage Description 是否许允此App使用蓝牙？
* 语音转文字权限：Privacy - Speech Recognition Usage Description 是否允许此App使用语音识别？
* 日历权限：Privacy - Calendars Usage Description 是否允许此App使用日历？
* 定位权限：Privacy - Location When In Use Usage Description 我们需要通过您的地理位置信息获取您周边的相关数据
* 定位权限: Privacy - Location Always Usage Description 我们需要通过您的地理位置信息获取您周边的相关数据

## 字体变大，原有的frame需要适配
程序内原有的2个字体的长度是24，现在2个子需要27宽度来显示

## 推送
如下图部分，不要忘记打开，所有的推送平台，不管是极光推送还是其它推送平台，要想收到推送，这个必须打开
![1](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20161009-1.png)

之后就可以收到推送了，iOS10推送的API也进行了更新，并且支持了语音，图片和自定义等功能

iOS10收到通知不再是在`[application: didReceiveRemoteNotification:]`方法去处理，iOS10推出新的代理方法，接受和处理各类通知(本地或者远程)

```
 - (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions))completionHandler { //应用在前台收到通知 NSLog(@"========%@", notification);
    }
    
 - (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void (^)())completionHandler { //点击通知进入应用 NSLog(@"response:%@", response);
   }

```

