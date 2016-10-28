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
		* App:
			
			* 处于前台，不会显示横幅，可通过 `didReceiveRemoteNotification `（ios7 before）,`didReceiveRemoteNotification:fetchCompletionHandler:`(ios7 after)获取通知内容
			* 处于后台，会展示横幅，无法获取通知内容
			* 处于退出，会展示横幅，无法获取通知内容
			* 点击图标启动，无法获取通知内容
			* 点击通知横幅启动，在`didFinishLaunchingWithOptions `获取通知内容
	* 通知内容类似如下:

	```
	{
  "_j_msgid" = 200806057;  // 第三方附带的 id，用于统计点击
  aps =     {
    alert = "显示内容";
    badge = 1;  // App 角标，可推送 n、+n、-n 来实现角标的固定、增加、减少
    sound = default;  // 推送声音，默认系统三全音，如需使用自己的声音，需要将声音文件拖拽&拷贝至 Xcode 工程目录任意位置，并在推送时指定其文件名
  };
  key1 = value1;  // 自定义字段，可设置多组，用于处理内部逻辑
  key2 = value2;
}
	```


* 后台推送

 	* 各种显示效果跟普通推送完全一样
 	* 必须携带`content-available` = 1
 	* 必须携带 alter ,badge ,sound 至少一个字段
 	* 仅 ios7以后支持
 	* 必须在Xcode工程中 `TARGETS – Capabilities – Background Modes – Remote notifications `开启该功能，具体可参照 [ iOS 7 Background Remote Notification](http://docs.jiguang.cn/client/ios_tutorials/#ios-7-background-remote-notification)


### App

* 处于前台，可通过`didReceiveRemoteNotification `(iOS7 before) ，didReceiveRemoteNotification:fetchCompletionHandler: （iOS 7 after）获取通知内容
* 处于后台，可通过` didReceiveRemoteNotification:fetchCompletion Handler:`获取通知内容，获取情况中于普通推送的唯一不同点，此时iOS系统允许开发者在App处于后台的情况下没执行一些代码，大概提供几分钟的时间，可以用来偷偷地刷新UI，切换页面，下载更新包等等操作
* 处于退出，无法获取通知内容
* 点击图标启动，无法获取通知内容
* 点击推送横幅启动，在`didFinishLaunchingWithOptions `获取通知内容

*通知内容类似如下:*

```
{
  "_j_msgid" = 2090737306;
  aps =     {
    alert = "显示内容";
    badge = 1;
    "content-available" = 1;  // 必带字段
    sound = default;
  };
  key1 = value1;
}
```


### 静默推送

* 没有任何展示效果
* 必须携带`"content-available" = 1`，因此静默必然是后台的
* 必须不携带 alert,badge,sound
* 可携带自定义字段

*App:*

* 处于前台，可以通过`didReceiveRemoteNotification `(iOS7 before),`didReceiveRemoteNotification:fetchCompletionHandler:` (ios 7 after)获取通知内容
* 处于后台，可通过`didReceiveRemoteNotification:fetchCompletion Handler:`获取通知内容，获取情况中与普通推送的唯一不同点，此时iOS系统允许开发者在App处于后台的情况下，执行一些代码，大概提供几分钟的时间，可以用来偷偷的刷新UI,切换页面，下载更新包等等操作
* 处于退出，无法获取通知内容

*通知内容类似如下:*

```
{
    "_j_msgid" = 3938587719;
    aps =     {
        alert = "";
        "content-available" = 1;  // 必带字段
    };
    key1 = value1;
}
```



## 推送目标篇

别名，标签，RegistrationID均是第三方提供的用于更方便地指定推送目标的功能

### Tip6:推送根据目标的不同可以分为:

* 广播
	* 无差别发送给所有用户
* 别名alias推送	
	* 第三方提供的功能
	* 一个手机的一款App只能设置一个 alias(可修改)
	* 建议对每一个用户都取不同的别名，以此来确定唯一的用户
	* 推送时可指定多个alias来发送同一内容
	* 仅指定alias的用户能够收到推送

* 标签tag推送
	* 第三方提供的功能
	* 可以设置多个，可增加，清空
	* 用于指定多样的属性，如 『1000』+『daily』+『discount』 可用于表示月消费超过 1k、喜欢购买日用品、偏好折扣商品的用户
	* 如果要删除，需要在上次设置时，将设置的tags保存至`NSUserDefaults`，本次剔除不需要的tag后，再重新设置
	* 推送时可指定多个tag来发同一个内容
	* 手机如果设置了推送指定的多个 tag 中任一个tag，都能够收到推送消息。如指定 『1000』+『globe』+『original』 （千元级消费者、全球购、原价），那么设置了 『100』+『globe』+『discount』（百元级消费者、全球购、折扣价）的用户可以收到该推送消息。
* Registration ID 推送

	* 第三方提供的功能
	* 在Tip 3的第三步时将`deviceToken `提供给第三方之后，其服务器会自动生成的指向该手机的唯一id
	* 可在推送时指定多个id来下发消息
	* 可用于对核心用户，旗舰用户的精准推送

	
## 应用消息篇

### Tip 7:应用内消息和推送通知的区别，消息:

* 不需要Apple推送证书
* 由第三方服务器下发，而不是APNS
* 相比通知，更快速，几乎没有延迟，可用于IM消息的即使传达
* 能够长时间保留离线消息，可获取所有历史消息内容
* 通过长连接技术下发消息，因此：
	* 手机必须启动并与第三方服务器简历连接
	* 如果手机启动立刻切换到后台，很可能连接没有建立
	* 手机必须处于前台才能收到消息
	* 手机从后台切回前台，会自动重新简历连接，并收到离线消息
* 没有任何展示（横幅，通知中心，角标，声音），因此可以：
	* 完全自定义字段实现UI效果
	* 完全在静默的情况下处理App内部逻辑
	* 使用一些App Store审核不会通过的功能，在审核时关闭功能，上架后通过接收消息，开启相关功能

## 组合大招

### Tip 8:tags的组合技巧

* 见 Tip5 - 标签tag 推送
* 可以再服务器端来统计分析用户行为，然后将指定的tags发送至手机，手机接收后再为用户打上对应的tags


### Tip 9:通知 + 消息的组合技巧

* 首先来看通知和消息的特性对比

XX |  通知  | 			消息
------------- |------------- | -------------
送达时间 | 可能存在几秒延迟  | 几乎没延迟
获取时机  | 处于前台或后台 能获取内容|仅处于前台能获取内容
离线内容| 保留一段时间，过期会抛弃，无法查询历史内容|始终保留，可查询全部历史内容
系统展示|会展示（静默推送或App处于前台不展示）|不展示



*由于各自的特性都存在差异，因此二者结合使用使得App推送性能最大化的必然选择：*

* 情景一

QQ/微信聊天，会同时下发一组通知+消息，如果用户没有启动QQ,虽有延迟但必须能够先收到通知，在收到通知的提醒之后，用户打开App,此时收到了离线消息，即使更新UI，与好友即使地发送/接收消息，（在收到通知后，断网，然后启动APP，你会发现此时手机里并不会显示刚刚通知的内容，因为它是依靠拉取消息来刷新页面的，而不是不够稳定的通知）










