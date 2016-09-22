---
layout: post
title: "iOS推送"
date: 2016-09-21 16:25:58 +0800
comments: true
categories: iOS
---

推送服务可以说是所有App的标配，不论是那种类型的App,推送都从很大程度上决定了App的打开率，使用率，存活率。因此熟知并掌握推送原理及方法,对每一个开发者来说都是必备技能，对每一个依赖App的公司都至关重要

从ios10新增的`UserNotifications Framework`可以发现，Apple整合了原有散乱的API,并且增加了很多强大的功能。以Apple官方的角度来看，也必然是相当重视推送服务对App的影响，以及对Apple生态圈长远发展的影响。
<!--more-->

##准备
### Tip 1：推送通知（Push Notification）必须购买Apple开发者账号，并使用特定的推送证书

* 使用免费账号不能推送
* 如果我们使用的是第三方推送服务，比如 <极光推送>,也必须购买开发者账号，因为所有的第三方都会将推送请求发至 `APNS`(Apple push Notification service，苹果推送通知服务)，所有的推送都是由Apns发送的
* 如果注册及正确的配置证书，参考这里[ios证书设置指南](http://docs.jiguang.cn/client/ios_tutorials/#ios_1)




## 原理
### Tip2:推送通知本身是iOS系统的行为，所以在App没有运行的时候：

* 仍然能够推送及接收(通知中心通知，顶部横幅，刷新App右上角小圆点等都会由系统控制和展示)
* 收到推送时，是无法再App的代码中获取到通知内容的。因为沙盒机制，此时App的任何代码都不可能被执行


### Tip3:手机向APNS注册推送服务

1. 在代码中注册推送服务:

```
 #ifdef __IPHONE_8_0
 if ([[UIApplication sharedApplication] respondsToSelector:@selector(registerUserNotificationSettings:)]) {
     UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge| UIUserNotificationTypeSound|UIUserNotificationTypeAlert categories:nil];
     [[UIApplication sharedApplication] registerUserNotificationSettings:settings];
 } else {
     UIRemoteNotificationType myTypes = UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound;
     [[UIApplication sharedApplication] registerForRemoteNotificationTypes:myTypes];
}
 #else
     UIRemoteNotificationType myTypes = UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound;
     [[UIApplication sharedApplication] registerForRemoteNotificationTypes:myTypes];
 #endif
```

2. 在第一次触发这段代码的时候，会有一个系统弹窗，询问你是否允许该App要给你推送信息。当你选择允许时，系统会打包App+手机唯一标识+证书信息 发送至Apns服务器注册推送服务，APNs系统会对该手机安装的该App是否有推送权限进行验证，所以必须要加入了Apple Device的手机，使用对应App的推送证书才能够成功注册。
3. 如果注册成功，则可以在`AppDelegate.swift`的如下方法中获取到`deviceToken`,它是对该手机+该App组合的一个唯一标识，当使用远程推送时，只需将推送消息发给指定的`deviceToken`即可使推送消息传达给指定手机的指定App上。因此如果你使用第三方，就需要在这里将`deviceToken`传给第三方。（在ios9为了更好的保护用户隐私，会出现多次重复删除/安装App导致`deviceToken`不断变化的情况。有时会出现一条推送手机会受到2次的问题，属于iOS9系统问题）

```
 -(void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {  
     [JPUSHService registerDeviceToken:deviceToken];//将 deviceToken 传给极光推送
 }
```

4. 如果以上步骤都成功，此时你能够获取到第三方提供的设备注册的id，能够获取到该id值，可以作为判断设备是否能够成功推送的标准，因此当你获取到该值时：

* 推送证书配置正确（你拥有了推送权限）
* 设备成功在Apns注册并返回了`deviceToken`（Apns能识别你的设备了）
* 返回的`deviceTOken`传给第三方，成功在第三方生成了唯一标识注册id（第三方能将你的设备信息传给APNS了）


5. 综上，注册及接受推送必须使用真机，必须连网


### Tip4:推送通知从服务端->App代码的过程

1. 使用你们公司或第三方的服务端向APNs发送推送请求（请参考苹果APNS相关资料，或者使用第三方提供的Rest Api）
2. APNS接受并验证推送请求
3. APNS找到设备下发推送
4. 手机收到推送通知，系统根据App状态进行处理：

* 前台收到：
	* 系统会将通知内容传到 `didReceiveRemoteNotification `
	
* 后台收到:

	* 如果开启了`Remote Notification`,系统将推送传到`didReceiveRemoteNotification:fetchCompletionHandler:`,否则此时代码中收不到推送
	* 展示横幅，通知中心，声音，角标
	
	
* 退出收到:

	* 如果点击推送横幅/通知中心而启动App,系统将通知传到`didFinishLaunchingWithOptions `
	* 展示横幅，通知中心，声音，角标

	
## 推送通知内容篇

### Tip5:推送通知分为本地/远程 2种类型:

* 本地通知，可指定推送时间，在该时间准时弹出推送通知。
* 远程推送通知，分为普通推送/后台推送/静默推送3种类型。存在延迟问题（由于Tip1的第2点，APNS的不稳定及高峰时段的巨量请求所致）

	* 普通推送
	
		* 就是我们在手机上平时见到的推送通知
		* 包括声音，横幅，角标，自定义字段












