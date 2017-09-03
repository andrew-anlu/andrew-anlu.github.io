---
layout: post
title: "xcode常见问题解析"
date: 2016-03-01 16:05:46 +0800
comments: true
categories: iOS
---

启动项目或者编译的时候，xcode总是有一些莫名的错误，现在我把遇到的问题和异常信息都记录下来，以此共勉



###CodeSign error: code signing is required for product type 'Application' in SDK 'iOS 9.2'

<!--more-->
解决方法:

* 去这个文件夹下 `/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/目标文件下` 比如我这个就是iPhoneOS9.2.sdk，
* 你需要拷贝这个文件 SDKSettings.plist  到桌面上
* 打开这个文件，找到DefaultProperties这个选项，设置里面的属性CODE_SIGNING_REQUIRED 为 NO
* 保存这个文件，再把这个文件拷贝到原始的路径下

搞定！
