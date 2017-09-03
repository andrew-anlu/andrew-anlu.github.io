---
layout: post
title: "Xcode更新后插件失效的解决方案"
date: 2016-04-22 16:17:45 +0800
comments: true
categories: Xcode
---

Xcode的插件对于开发者来说无疑是一把利器，让开发者能够将更多的时间和精力放在代码上面。但是开发者都会遇到一个问题，就是每次Xcode更新到最新的版本，之前的插件全部都失效了，需要重新安装一遍很是麻烦。

<!--more-->
插件失效的原因:

>  
>> * 系统安装了一个Xcode
>> * 开发者未正确的将自己的DVTPlugInCompatibilityUUID添加到插件中
>> * 成功安装了插件，但是却在Xcode识别插件的时候，选了 Skip Bundle
>>
>
>
>


##终端指令实现

//获取DVTPlugInCompatibilityUUID字段

```
defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID

```

##找到插件的目录

1. 打开xcode插件所在的目录:`~/Library/Application Support/Developer/Shared/Xcode/Plug-ins`,打开路径的快捷键是shift＋command＋g 然后输入上面的地址
2. 选择已经安装的插件，例如:VVDocumenter-Xcode，右键显示包内容
3. 找到info.plist文件，找到DVTPlugInCompatibilityUUIDs的项目，添加一个Item,value的值为之前的Xcode的UUID,(或者直接在item0中value修改也可以)保存

![step2](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160422-1.png)


##重启xcode
重启Xcode时会提示“Load bundle”、 “Skip Bundle”，这里必须选择“Load bundle”，不然插件无法使用

![step1](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160422-0.png)



##效果如下
![s](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160422-2.png)






