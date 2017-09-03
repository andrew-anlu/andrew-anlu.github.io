---
layout: post
title: "如何用cocoaPod发布框架"
date: 2016-03-15 14:54:27 +0800
comments: true
categories: cocoapods
---

##前言
[cocoapods](https://cocoapods.org/)是一款强大的框架依赖构建的工具，类似于java中的maven,可以快速搭建你项目中需要的框架，并且它可以自动帮你关联好第三方库之间的依赖关系， 这个很像ubuntu下的apt安装软件，自动帮你下载需要的依赖。
<!--more-->


使用cocoapod可以轻松的创建框架，发布到github上，可以让全世界的开发者下载到你的框架，并且使用，如果能为全世界的开源社区做点贡献，也是一件非常兴奋的事情。

开发静态库有两种方法:

##手动创建框架
手动创建比较繁琐，而且容易出现问题，大致步骤如下:

1. 在xcode中创建一个 `Cocoa Touch Static Library`
2. 创建 `Podfile`文件
3. 执行 `pod install`完成整个项目的构建
4. 如果需要demo,手动创建示例程序，使用pod添加对私有静态库的依赖，重复执行 `pod install`来完成框架的构建

##使用Pod创建框架
`要求你系统中安装的cocoapod的版本>0.3`
电脑环境:

1. OS X ElCapitan 10.11.1
2. cocoapod 0.39
3. xcode 7.2
4. ruby 2.0.0p645 (2015-04-13 revision 50299) [universal.x86_64-darwin15]
5. Homebrew 0.9.5 (no git repository)


可以通过pod官网提供的命令来创建，让我们快速开始:
	
	pod lib create podLib-Andrew
这行命令会自动从pod官网的仓库中下载模板
然后它会问你几个问题

1. What language do you want to use?? [ ObjC / Swift ] =>看你需要
2. Would you like to include a demo application with your library? [ Yes / No ]=>是否包含一个demo，我一般Yes
3. Which testing frameworks will you use? [ Specta / Kiwi / None ] =>是否需要包含一个测试框架
4. Would you like to do view based testing? [ Yes / No ]
5. What is your class prefix? =>类的前缀是什么


执行成功之后，会自动用xcode打开工程，如果你的pod版本太新，比如1.0beta版本，可能不会自动创建workspace工作目录，这是由于pod1.0有些语法变了，你要相应的更改Podfile文件中的语法，具体参考官方最新的语法。

最终生成的目录结构如下:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160315-0.png)

1. Example 你的实例代码文件夹
2. .travis.yml -持续集成的配置文件 [travis-ci](https://travis-ci.org/)
3. _Pods.xcproject 工程的一个连接，类似快捷方式
4. LICENSE 默认是 [MIT License](http://en.wikipedia.org/wiki/MIT_License)
5. README.md 描述文件
6. pod 你存储源代码的地方


##设置你的框架
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/xcode.png)

让我们看打开后的工程目录

1. 你可以编辑Podspec metadata，包括里面的版本号和介绍等


```
#
# Be sure to run `pod lib lint podLib-Andrew.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see http://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = "podLib-Andrew"
  s.version          = "0.0.1"
  s.summary          = "这是我的第一个框架，希望能够帮到你"

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!  
  s.description      = <<-DESC
                        "这是我的第一个框架，希望能够帮到你"
                       DESC

  s.homepage         = "https://github.com/andrew-anlu/podLib-Andrew"
  # s.screenshots     = "www.example.com/screenshots_1", "www.example.com/screenshots_2"
  s.license          = 'MIT'
  s.author           = { "Andrew" => "anluanlu123@163.com" }
  s.source           = { :git => "https://github.com/andrew-anlu/podLib-Andrew.git", :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.platform     = :ios, '7.0'
  s.requires_arc = true

  s.source_files = 'Pod/Classes/**/*'
  s.resource_bundles = {
    'podLib-Andrew' => ['Pod/Assets/*.png']
  }

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'
end

```

* s.name是我们库的名称
* s.version是库原代码版本号
* s.summary是对我们库的一个简单的介绍
* s.homepage声明库的主页
* s.license是所采用的授权版本
* s.author是库的作者
* s.platform是我们库所支持的软件平台，这在我们最后提交进行编译 时有用
* s.source声明原代码的地址.

按照默认配置，类库的源文件将位于Pod/Classes文件夹下，资源文件位于Pod/Assets文件夹下，可以修改s.source_files和s.resource_bundles来更换存放目录。s.public_header_files用来指定头文件的搜索位置。
s.frameworks和s.libraries指定依赖的SDK中的framework和类库，需要注意，依赖项不仅要包含你自己类库的依赖，还要包括所有第三方类库的依赖，只有这样当你的类库打包成.a或.framework时才能让其他项目正常使用.

##建立远程版本库
现在让我们去github上建立一个仓库，名字和工程的名字保持一致`podLib-Andrew`
![pic](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160315-1.png)

切换到终端，进入刚才的工程的根目录

```
//添加到本地
git add . 

//提交到本地仓库
git commit -m "initial commit"

//和远程仓库建立关联关系
git remote add origin git@github.com:andrew-anlu/podLib-Andrew.git

//执行更细远程仓库的数据
git pull origin master

//重新执行添加
git add .

git commit -m "commit"

//提交代码到远程仓库
git push origin master

```

##编写源代码
在Pod的classes文件夹中新建一个类 `StringUtils`
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160315-2.png)

StringUtils.h

```
#import <Foundation/Foundation.h>

@interface StringUtils : NSObject
+(void)sayHello;
@end

```

StringUtils.m

```
#import "StringUtils.h"

@implementation StringUtils
+(void)sayHello{
    NSLog(@"Hello world");
}
@end

```

注意源代码的存放位置一定要和 .podspec中的 s.source 路径要一致，不然不会起作用的。
运行 `pod install`,也许你也注意到了，每次添加或者修改类，或者添加图片等操作，都要执行 `pod install`或者`pod update`来更新应用。

>*注意*
>如果是生成swift功能，在Podfile文件中一定要有 `use_frameworks!` ，这行命令是指定框框是动态框架，因为swift要求要么依赖全部是动态框架，要么全部不是，所以如果注释掉，会报编译报错；
>
>另外，swift工程中的源文件，如果想让外部调用的话，一定要声明成**`public`**,
>这一点很重要。

###Note
添加完源代码之后，如果要在demo工程中引用，发现是引用不到的，原因就是你必须要把类库加入到demo工程中才行，默认在Podfile文件中生成的类库是动态类库，即 .framework的。
我尝试了好久，一直引用不了，于是就改成生成.a静态库的方式，完美引用。

修改成.a类库的方法就是在 Podfile中，把`#use_frameworks!` 注释掉就行了，这行代码默认就是生成动态类库的。
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160317-0.png)

###加入持续集成(Travis CI)
这个模板包含了一个 `.travis.yml `文件，加入你在Github上开关了你的框架，打开 [travis-ci官网](https://travis-ci.org/)，并且用github账号登录并且同步，打开你要开源的项目，设置 on

![lgogo](http://7xkxhx.com1.z0.glb.clouddn.com/travi.png)

###提交本地代码库
修改podLib-Andrew.podspec 中的 `s.source`,

	  s.source           = { :git => "https://github.com/andrew-anlu/podLib-Andrew.git", :tag => s.version.to_s }

提交代码，并要打上tag

```
git add .
git commit -m "commit v0.0.1"
git tag 0.0.1
git tag -a 0.0.1 -m "v0.0.1"
```



###验证类库
开发完静态类库之后，需要运行 `pod lib lint`验证一下类库是否符合 pod的要求。`pod spec lint`也是用来验证是否有效，区别在于

1. pod lib lint  不访问网络，只在本地验证
2. pod spec lint 检查远程仓库和远程的tag

pod官方推荐用[trunk](https://guides.cocoapods.org/making/getting-setup-with-trunk)来发布框架，[trunk入门](https://guides.cocoapods.org/making/getting-setup-with-trunk)

假如你只想发布一个私人的框架，请看这个教程 [Private Specs Repos](https://guides.cocoapods.org/making/private-cocoapods),如果你想发布一个已经存在的私有框架，可以用下面的命令

	pod repo push SPEC_REPO *.podspec --verbose


##发布框架

	pod trunk push podLib-Andrew.podspec


如果发布成功，执行 `pod search podLib`
就会搜到你的框架

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160315-4.png)

#Done
👍


