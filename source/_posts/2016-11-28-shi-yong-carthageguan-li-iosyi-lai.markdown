---
layout: post
title: "使用Carthage管理ios依赖"
date: 2016-11-28 15:55:02 +0800
comments: true
categories: swift
---

[Carthage](https://github.com/Carthage/Carthage.git )官网已经针对cocopods和carthage进行了详细的说明：

<!--more-->

首先，cocoaPods会直接创建和修改项目的workspace配置，一切都是为了便捷，我们只需要修改pod文件并不需要过多的关心其它事情，CocoaPods创建的也是高度集成的项目。而Carthage的特点是灵活，耦合度不高，集成时不需要集成相应的project,不需要创建workspace,而仅仅需要依赖打包好的framework文件。

其次，CocoaPods相对来说功能要比Carthage多很多，在国内由于墙的原因，我们都改成了淘宝的源来更新CocoaPods,相信我，如果你不翻墙，很多东西不能用，更新不下来，版本错误等一系列原因会让你不得不放弃一起看起来非常好用的第三方库。而Carthage似乎只需要从github上下载项目即可，配置更是简单，使用的项目项目干干净净，所有的第三方库就像苹果原生的framework一样美好，从此你不再需要担心CocoaPods的库用不了，不用花大量时间去修复用CocoaPods打包时出现的各种问题了，如果你用过CocoaPods，当你开始使用Carthage的时候，你会爱上这个工具的。


## 安装使用Carthage

假如你的电脑上已经安装了Homebrew,打开终端，输入如下命令:

```
$ brew update

$ brew install carthage
```

如果你不喜欢使用终端，也可以从网站`https://github.com/Carthage/Carthage/releases`下载最新版的Carthage.pkg来更新。

现在，你已经安装好了Carthage,接下来就是在你的项目中使用carthage了：

1. 通过终端进入到项目所在的文件夹

```
$ cd ~/Path/Project
```

后面的路径替换成你的项目所在的路径

2. 创建一个空的carthage文件：

```
$ vim Cartfile
```
此时在你的项目文件夹里会创建一个名为Cartfile的文件

3. 在Cartfile文件里输入

```
github "Alamofire/Alamofire" ~> 3.0

github "SwiftyJSON/SwiftyJSON"
```

上面仅仅是举个栗子
，如果您是用命令行打开的，按下ESC,输入 :wq，保存关闭当前窗口

### 版本的含义
~> 3.0 表示使用版本3.0以上但是低于4.0的最新版本，如3.5，3.7...

== 3.0表示使用3.0版本


`>=` 3.0表示使用3.0或者更高的版本
如果你没有指明版本号，则会自动使用最新的版本

4. 在终端执行命令

```
$ carthage update --platform iOS
```


carthage会为你下载和编译所需要的第三方库，当命令执行完毕，在你的项目文件夹中会创建一个名为Carthage的文件夹


在 ~/Carthage/Build/iOS里会出现xxx.framework文件已经为你创建好了

5. 现在打开你的项目，点击Project,选择target,点击`+`,将刚才生成的framework文件拖到` Linked frameworks and Binaries`里；

同样也要把生成的framework拖到`Embedded Binaries`中



## 开始使用

如果你已经导入framework成功了，然后使用：

```
import Alamofire

import SwiftyJSON
```


Please enjoy it!