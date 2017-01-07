---
layout: post
title: "iOS中的热修复"
date: 2016-11-07 16:46:54 +0800
comments: true
categories: iOS
---


## 背景需求

### 为什么我们需要热修复

* 工作中容易犯错，bug难以避免
* 开发和测试人力有限
* 苹果AppStore审核周期太长，一旦出现严重bug难以快速上线新版本



## JSPatch简介

JSPatch诞生于2015年5月，最初是腾讯广研高级ios开发@bang的人格项目。它能够使用JavaScripit调用Objective-C的原声接口，从而动态植入代码来替换旧代码，以实现修复线上bug.

## JSPatch与wax对比

![1](http://upload-images.jianshu.io/upload_images/611240-3d1af75ebfe7de01.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最关键的是JSpath可实现方法粒度的线上代码替换，能修复一切代码引起的bug.而Wax无法实现。

## JSPatch实现原理

### 基础原理

Objective-C是动态语言，具有运行时特性，该特性可通过类名称和方法名的字符换获取该类和该方法，并实例化调用


```
Class class = NSClassFromString(“UIViewController");
id viewController = [[class alloc] init];  
SEL selector = NSSelectorFromString(“viewDidLoad");
[viewController performSelector:selector];
```

也可以替换某个类的方法为新的实现：

```
static void newViewDidLoad(id slf, SEL sel) {}
class_replaceMethod(class, selector, newViewDidLoad, @"");
```

还可以注册一个类，为类添加方法：

```
Class cls = objc_allocateClassPair(superCls, "JPObject", 0);
objc_registerClassPair(cls);
class_addMethod(cls, selector, implement, typedesc);
```

### JavaScript调用
我们可以用JavaScript对象定义一个Objective-C类：

```
{
  __isCls: 1,
  __clsName: "UIView"
}
```

在OC执行JS脚本前，通过正则把所有方法调用都改成__c()函数，再执行这个JS脚本，做到了类似OC/Lua/Ruby等的消息转发机制：

```
UIView.alloc().init()
->
UIView.__c('alloc')().__c('init')()
```

给JS对象基类Object的prototype加上c成员，这样所有对象都可以调用到c,根据当前对象类型判断进行不同操作：

```
Object.prototype.__c = function(methodName) {
  if (!this.__obj && !this.__clsName) return this[methodName].bind(this);
  var self = this
  return function(){
    var args = Array.prototype.slice.call(arguments)
    return _methodFunc(self.__obj, self.__clsName, methodName, args, self.__isSuper)
  }
}
```

## 互传消息

JS和OC是通过JavaScriptCore互传消息的。OC端在启动JSPatch引擎会创建一个JSContext实例，JSContext是js代码的执行环境，可以给JSContext添加方法。JS通过调用JSContext定义的方法把数据传给OC,OC通过返回值传回给JS.调用这种方法，它的参数/返回值 javaScripotCore都会自动转换，OC里的NSArray,NSdictionary
,NSString,NSNumber,NSBlock会分别转为JS端的数组/对象/字符串/数字/函数类型  对于一个自定义ID对象，JavaScriptCore会把这个自定义对象的指针传给JS,这个对象在JS无法使用，但在回传给OC时OC可以找到这个对象。对于这个对象声明周期的管理，如果JS有变量引用时，这个OC对象引用计数就加1，JS变量的引用释放了就减一，如果OC上没别的持有者，这个OC对象的生命周期就跟着JS走了，会在JS进行垃圾回收时释放。


## 方法替换

1. 把UIViewContrller的 `-viewWillAppear:`方法通过`class_replaceMethod()`接口指向`_objc_msgForward `,这是一个全局IMP,OC调用方法不存在时都会转发到这个IMP上，这里直接把方法替换成这个IMP,这样调用这个方法时就会走到` -forwardInvocation:`
2. 为UIViewController添加`-ORIGviewWillAppear:`和`-_JPviewWillAppear:`两个方法，前者指向原来的IMP实现，后者是新的实现，稍后会在这个实现里回调JS函数
3. 改写UIViewController的`-forwardInvocation:`方法为自定义实现。一旦OC里调用UIViewController的`-viewWillAppear:`方法，经过上面的处理会把这个调用转发到`forwardInvocation:`，这时已经组装好了一个NSInvocation,包含了这个调用的参数。在这里把参数从NSInvocation反解出来，待着参数调用删除新增加的方法`-JPviewWillAppear:`,在这个新方法里获取到参数传给JS,调用JS的实现函数，整个调用过程就结束了，整个过程图示如下：

![1](http://upload-images.jianshu.io/upload_images/611240-d079409a185f394c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后一个问题，我们把UIViewController的`-forwardInvocation:`方法的实现给替换掉了，如果程序里挣得有用到这个方法对消息进行转发，原来的逻辑怎么办？首先我们在替换`-forwardInvocation:`方法前会新建一个方法`-ORIGforwardInvocation:`，保存原来的实现IMP，在新的`-forwardInvocation:`实现了做个判断，如果转发的方法是我们想改写的，就走我们的逻辑，若不是，就调用`-ORIGforwardInvocation:`走原来的流程

## JSPatch代码示例

jspatch在oc上的调用十分简单

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions { 
[JPEngine startEngine]; 
NSString *sourcePath = [[NSBundle mainBundle] pathForResource:@"demo" ofType:@"js"]; 
NSString *script = [NSString stringWithContentsOfFile:sourcePath encoding:NSUTF8StringEncoding error:nil]; 
[JPEngine evaluateScript:script];
}


```

一个JavaScript修复Objective-C的bug的示例:

```
@implementation JPTableViewController

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
  NSString *content = self.dataSource[[indexPath row]];  //可能会超出数组范围导致crash
  JPViewController *ctrl = [[JPViewController alloc] initWithContent:content];
  [self.navigationController pushViewController:ctrl];
}

@end
```

上述代码中取数组元素出可能会超出数组范围导致crash.如果在项目里引用了JSPatch,就可以发JS脚本修复这个bug:

```
defineClass("JPTableViewController", {
  tableView_didSelectRowAtIndexPath: function(tableView, indexPath) {
    var row = indexPath.row()
    if (self.dataSource().length > row) {  //加上判断越界的逻辑
      var content = self.dataArr()[row];
      var ctrl = JPViewController.alloc().initWithContent(content);
      self.navigationController().pushViewController(ctrl);
    }
  }
}, {})
```


## 热修复的解决方案

1. 版本更新策略

* 考虑到下一个提交的App版本已经修复了上一个版本的bug,所以不同的App版本对应的补丁肯定也不同，同一个App版本下，可以出现递增的补丁版本
* 补丁为全量更新，即最新的版本补丁包括旧版的补丁的内容，更新后新版补丁覆盖旧版补丁
* 补丁分为可选补丁和必选补丁，必选补丁用于重大bug的修复，如果不更新必须补丁则App无法继续使用。如下图2中，补丁版本v1234对应各自版本的用户，补丁v3为必须更新，补丁v1,v2,v4为可选补丁，则v1,v2必须更新到v4才可使用；而v3的哟过户可先使用，同事后台静默更新到v4

## 安全策略
安全问题在于JS脚本可能被中间人攻击替换代码。可采取一下三种方法

1. 对称加密： 如zip的加密压缩，Aes等加密算法。优点是简单，缺点是安全性低，易被破解。若客户端被反编译，密码字段泄露，则完全破解。
2. HTTPS:优点是安全性高，证书在服务器未泄露，就不会被破解，缺点是部署麻烦，如果服务器本来就支持Https,使用这种方案也是一种不错的选择。
3. RSA校验：安全性高，部署简单

![1](http://upload-images.jianshu.io/upload_images/611240-14723080a9823ced.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

详细校验步骤如下：

1. 服务器计算出脚本文件的MD5值，作为这个文件的数字签名
2. 服务器通过私钥加密算出的MD5值，得到一个加密后的md5值
3. 把脚本文件和加密后的md5值一起发给客户端
4. 客户端拿到加密后的md5值，通过保存在客户端的公钥解密
5. 客户端计算脚本文件的md5值
6. 对比第 4/5 步的两个md5值(分别是客户端和服务器端计算出来的MD5值)，若相等则通过校验

## 客户端策略

客户端具体策略如下图：

![1](http://upload-images.jianshu.io/upload_images/611240-3f5d0d89e0b3833d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 用户打开App时，同步进行本地补丁的加载
2. 用户打开App时，后台进程发起异步网络请求，获取服务器中当前App版本所对应的最新补丁版本和必须的补丁版本
3. 获取补丁版本的请求回来后，跟本地的补丁版本进行对比
4. 如果本地补丁版本小于必须版本，则提示用户，展示下载补丁界面，进行进程同步的补丁下载。下载完成后重新加载App和最新补丁，再进入App
5. 如果本地补丁版本不小于必须版本，但小于最新版本，则进入App,不影响用户操作。同时进行后台进程异步静默下载，下载后补丁保存在本地，下次App启动时再加载最新补丁。
6. 如果版本为最新，则进入App