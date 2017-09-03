---
layout: post
title: "iOS的后台模式"
date: 2016-12-23 14:32:36 +0800
comments: true
categories: swift
---


回到2010年ios4时代，苹果就已经推出了多任务.

<!--more-->

用户和开发者们经常抱怨ios的多任务系统允许做什么呢

苹果已经限制了用户使用后台操作，也许这些操作是用来提高用户体验的，或者是延长电池的使用寿命。

你的App仅仅是允许在后台运行而已。在一些特殊的情况下,这些包括:播放音频，获取地理位置，从服务器获取最新的数据。

如果你的任务不是这些类别中的一个，那么后台模式不适合你。

如果你利用后台模式做了一些其它的工作或者任务来欺骗系统，在你提交AppStore时，有可能会被拒绝上架，所以要慎重。


## 起步
在开始工程之前，你可以快速浏览一下看看后台模式能够做什么？在xcode8中，你可以看到在你的app target中，有个一`Capabilities `的tab页，打开之后有一个列表:

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/EnableBackgroundCapability-650x355.png)

1. 在`Project Navigato`中选择一个工程
2. 选择app target
3. 选择 `Capabilities `tab
4. 打开`Background Modes`开关

在后台模式中，你将要研究4中方式来做后台进程的工作:

* 播放音频:在app进入后台模式中，仍然能够播放音频文件
* 地理坐标更新:当设备坐标位置该改变的时候，这个App能够更新坐标
* 加载一定长度的任务:在`whatever`的案例中，App在有限的时间内，可以运行任意的代码
* 后台获取数据:定时更新内容

如果你仅仅是对其中一个或者多个感兴趣，你可以跳过其它的介绍；

[下载开始工程](https://koenig-media.raywenderlich.com/uploads/2016/09/TheBackgrounder-rev3-Starter.zip),

编译运行工程，你会看到有4个tab,每个代表一个后台模式:

![2](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-StarterProjectTabs.png)

>注意:
>
>为了展现全部的效果，你将要使用真正的设备去测试，经验之谈，假如你忘记去设置配置文件，这个app在模拟器的后台模式下是一直运行的。然而，当你切换到真机的时候，它将不再工作。所以最好用真机去测试。
>
>


## Playing Audio


首先，音频播放。

这里有几种方式在IOS设备上去播放音频，大部分他们要求提供回调的实现方法去提供更多的音频数据去播放、

如果你想从一个二进制文件中播放一个音频，你可以连接网络，在连接后的回调处理方法中提供持续的音频数据。

当你激活Audio进入后台模式后，你的ios设备将会继续回调，甚至你的app不是当前激活的状态。这就对了--这个 audoi background模式是一个自动虚拟的，你仅仅需要去激活它并且提供基础实施去适当的处理它.


在这一章中，你将要回顾下app播放器，验证下后台模式是否工作正常，确保这个audio可以在后台模式下工作的能力，证明他是可以在工作的


打开`AudioViewController.swift`文件，

这个App使用了`AVQueuePlayer `去顺序播放歌曲，这个View Controller监听播放器的`currentItem`.

当这个app是处于激活状态的时候，这个music title label将会显示，当这个app处于后台模式中的时候，它将在控制台打印出标题，这个文本显示信息在后台模式的时候将会更新。重点是当你的app在后台的时候，是否能继续收到回调信息。

编译运行你的app:
![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-AudioScreen.png)

现在点击`Play`,音乐就会开始了.

测试下后台模式，如果你是在真机上测试的，请点击Home按钮，此时音乐停止了，为什么呢?这里还有一件重要的事情没有做

对于大多数的后台模式而言，（"whatever"模式是个特殊），你需要确保在进入后台模式后，你的代码依然能够运行。

回到xcode,做下面的事情:

1. 点击工程的`Project Navigator`
2. 点击`TheBackgrounder `target
3. 点击`Capabilities `标签页
4. 找到`Background Mode`并且打开开关
5. 选择`Audio, AirPlay and Picture in Picture.`

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-EnableAudioInBG.png)

再次编译运行，重复刚才的操作，此时在后台模式下，你就能听到音乐继续在播放了。

你应该能在控制台看到这个更新时间，证明你的代码甚至在后台模式下依然在工作

wow,如果你已经有一个音乐播放器了，在后台模式下播放是很简单的。


## 收到地址坐标更新

当你的定位是在后台模式下，你的app将会收到用户地址定位的更新的代理消息，在后台模式下，你可以控制定位更新的精度，甚至可以改变这个精度。

第二个tab页就是关于定位更新的。打开`LocationViewController.swift`,和audio example例子很像，这个定位更新的后台模式是很容易去实现的，假如你之前做过定位相关的工作。

在这个控制器里，你将会找到`CLLocationManager `，为了收到定位消息你需要配置`CLLocationManager `实例，在这个案例中，当你打开屏幕上的UISwitch开关，你的app定位监听就会打开，定位服务将会收到App在地图上放置的大头针。当你的app是在后台模式下的时候，你将会看到在控制台上看到有日志输出。

在`CLLocationManager `实例中，一行重要的代码是调用`requestAlwaysAuthorization ()`,这是一个请求权限的提示在ios8后，它会弹出一个权限的提示框，并且在后台收到定位信息。

这个同样需要在xcode中进行设置，选择`Location updates `，让ios系统知道你的app在后台模式下想要继续收到定位更新的信息

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-EnableLocationInBG.png)

在选择了刚才的checobox之后，ios8要求你必须在你的`info.plist`中设置一个key,去解释用户为什么需要在后台模式下进行定位更新，如果你不做这个设置，你的定位请求将会失败。

* 在xcode中选择一个工程
* 在`TheBackgrounder `target中，选择info tab 页
* 选择已经存在的行
* 点击`+`按钮去添加一个新key
* 添加key的名字为:`Privacy – Location Always Usage Description`
* type -> string

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-AddLocationPrivacyMessage.png)

现在，编译运行你的工程，把页面上的switch开关打开

当你是第一次操作的时候，你将会收到一个弹出的提示框，是关于你的定位权限的，点击`Allow`,并且步行走出你所在的大楼或者建筑物，你将会看到你的定位正在更新，甚至在模拟器上也会看到

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-AllowLocationAccess.png)

过一会，你将会看到如下:
![2](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-LocationUpdatesOnMap.png)

如果你的app是在后台模式下，你将会看到app更新坐标的日志信息。再次打开你的app,你将会发现你的地图上已经为你的地址坐标布满了大头针。

如果你是用模拟器进行测试的，你可以这样设置:`Debug \ Location menu:`


![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-DebugLocationSimulation.png)

尝试设置location为`Freeway Drive`,然后点击home按钮，你将会看到驾驶在加利福利亚高速公路上的路线的坐标信息，以及你的进度信息都会作为日志打印到控制台上。


```
App is backgrounded. New location is %@ <+37.33500926,-122.03272188> +/- 5.00m (speed 7.74 mps / course 246.09) @ 9/5/16, 10:20:07 PM Mountain Standard Time
App is backgrounded. New location is %@ <+37.33500926,-122.03272188> +/- 5.00m (speed 7.74 mps / course 246.09) @ 9/5/16, 10:20:07 PM Mountain Standard Time
App is backgrounded. New location is %@ <+37.33500926,-122.03272188> +/- 5.00m (speed 7.74 mps / course 246.09) @ 9/5/16, 10:20:07 PM Mountain Standard Time
```


是不是很容易，让我们点击第三个tab,开始学习第三种后台模式

## 请求有限时间的任务

这个后台模式是被叫做[ Executing a Finite-Length Task in the Background](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW3)

技术上说，这个算不上后台模式，你不需要在你的app上设置后台模式的类型在`Capabilities `菜单中，代替的是，它仅仅是一个api,当你的app处于后台模式下的时候，运行任意的代码。


在过去，这个模式经常用来完成上传和下载的工作，系统大概提供10分钟来完成这些任务。

但是如果网络很慢，进程不能按时完成怎么办呢？这会让你的app很尴尬，你不得不做些错误的处理工作使得事情看起来合乎情理。因为这个原因，苹果引进了`NSURLSession `

尽管`NSURLSession `不是后台模式这个主题的介绍范围之内，但是`NSURLSession `具有稳健的后台处理机制，假如你正在下载一个很大的文，甚至设备重启之后，它都有下载文件完的强大的能力。如果你要学习NSURlSession，[请查看这个教程](https://www.raywenderlich.com/110458/nsurlsession-tutorial-getting-started)

这个案例介绍的后台模式是：当你运行很长的任务需要长时间的后台执行代码，比如渲染或者写一个视频文件到相机里面....这些都是很耗时的操作

![1](https://koenig-media.raywenderlich.com/uploads/2013/04/Whatever_cheers.png)

这个当然仅仅是一个例子，你的代码可以是任意的，你可以使用这些api去做更多美好的事情：执行很冗长的运算，过滤图片，渲染3D效果....whatever!

你的想象是有限的，无论什么都是可以做的。

你的app在后台运行的时间长短是靠ios系统决定的，不是你我能左右的。这里不能保证能完成指定的工作，但是你可以通过`UIApplication `的`backgroundTimeRemaining `属性来检查工作。这个将会告诉你在后台运行了多久.

一般来说，通过观察者模式你只有3分钟的时间，再次，这个不能保证这个api给你一个准确的时间数字-所以你不经仅仅依赖这个数字，你可能得到的结果是5分钟也有可能是5秒，所以你的app可以准备做些什么....


打开`WhateverViewController.swift`，这个控制器将要书序计算Fibonacci 数字，并且展示这个结果。假如你挂起这个app在这个设备上，这个计算将要停止，当这个app再次被激活的时候，数字才会再次收集。

你的任务就是创建一个后台任务，当程序进入后台的时候依然能够运行计算


你首先要添加一个属性在`WhateverViewController `中:

```
var backgroundTask: UIBackgroundTaskIdentifier = UIBackgroundTaskInvalid

```
这个属性是被用于在后台模式下任务的identify.

下面添加下面的方法在`WhateverViewController `

```
func registerBackgroundTask() {
  backgroundTask = UIApplication.shared.beginBackgroundTask { [weak self] in
    self?.endBackgroundTask()
  }
  assert(backgroundTask != UIBackgroundTaskInvalid)
}
 
func endBackgroundTask() {
  print("Background task ended.")
  UIApplication.shared.endBackgroundTask(backgroundTask)
  backgroundTask = UIBackgroundTaskInvalid
}
```

`registerBackgroundTask()`告诉iOS你需要更多的时间去计算，无论你是在后台做什么工作。当这个调用过后，如果你的app是在后台，它就会需要获取CPU的时间，直到你调用`endBackgroundTask()`去终止后台任务


好了，如果在后台运行了一段时间后，你不调用`endBackgroundTask() `，iOS将要调用默认的闭包`beginBackgroundTask(expirationHandler:)`去给你一个机会去执行停止的代码。所以，这是一个非常好的地方去调用` endBackgroundTask() `，告诉iOS系统你做完工作了。

如果你不这样做，继续执行代码，那么你的app将会被强行终止


现在，你需要更新`didTapPlayPause(_:) `中的代码，去注册后台模式并且结束它。这里有两行注释的代码，你需要在下面添加一些代码:

调用`registerBackgroundTask()`在`register background task`注视下:

```
// register background task
registerBackgroundTask()
```

当计算开始的时候`registerBackgroundTask() `将会被调用，所以你能继续在后台模式下计算

现在，在`end background task`注视下添加下面代码:

```
// end background task
if backgroundTask != UIBackgroundTaskInvalid {
  endBackgroundTask()
}
```

现在当你不再需要额外的CPU时间的时候，你可以调用`endBackgroundTask()`去停止计算。

你每次调用`beginBackgroundTask(expirationHandler:). `方法的时候，再次调用`endBackgroundTask()`，这是很重要的。

如果你调用`beginBackgroundTaskWithExpirationHandler(_:) `两次，而调用`endBackgroundTask()`仅仅一次。你此时依然能够获取到CPU时间，直到你在第二个后台任务中第二次调用了`endBackgroundTask()`.

这也就是为什么你需要`backgroundTask `


现在更新`calculateNextNumber()`方法，添加两个展现`application’s `状态的判断:

```
switch UIApplication.shared.applicationState {
case .active:
  resultsLabel.text = resultsMessage
case .background:
  print("App is backgrounded. Next number = \(resultsMessage)")
  print("Background time remaining = \(UIApplication.shared.backgroundTimeRemaining) seconds")
case .inactive:
  break
}
```

这个label将要在Application是激活状态的时候更新，当你这个applicaton是在后台模式的时候，消息将会被打印。

说明新的计算结果是什么，并且进入后台模式多长时间

编译运行,点击第三个tab


![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-RunningWhatever.png)

点击`Play`按钮你将要看到app的计算结果，现在点击home按钮，然后观察xcode的控制台，你将要看到app依然在更新数字。

大部分时间，这个时间将会开始在180(180秒=3分钟)，下降在5秒。

如果你等待这个时间去过期，当你到达5秒的时候（根据条件的不同，可能会到达另外一个值）,这个过期block将会被调用，你的app将会什么都打印而停止。此时，当你返回App,这个时间计时器将会再次开启，整个行为将会继续。


这个有一个bug，假设你的app是在后台模式，等待直到系统分配的时间过期。在这个案例中，你的app将要调用过期的回调函数`endBackgroundTask()`,因此结束需要后台时间。

如果这个时候你返回App,这个time将要继续开启，你拿上离开这个App(进入后台)，你将不会获取后台执行的时间。

为什么？因为没有在过期和返回后台模式之间 调用`beginBackgroundTaskWithExpirationHandler(_:)`

如何解决这个问题呢？这里有几个方式去解决，其中之一就是使用状态改变的时候发送通知


修复这个bug,首先，添加一个新的方法命名为`reinstateBackgroundTask().`

```
func reinstateBackgroundTask() {
  if updateTimer != nil && (backgroundTask == UIBackgroundTaskInvalid) {
    registerBackgroundTask()
  }
}
```

当进入后台模式并且后台任务不是不可用的状态，&时间计时器不是nil,你需要重置这个状态。在这个案例中，你仅仅需要去调用`registerBackgroundTask().`


现在覆盖` viewDidLoad()`，添加如下代码:

```

override func viewDidLoad() {
  super.viewDidLoad()
  NotificationCenter.default.addObserver(self, selector: #selector(reinstateBackgroundTask), name: NSNotification.Name.UIApplicationDidBecomeActive, object: nil)
}
```

这个新设计的方法就是当你Application再次激活的时候调用

当你订阅这个通知的时候，你应该想到什么时候去取消订阅通知，使用 析构函数`deinit`去实现。

```
deinit {
  NotificationCenter.default.removeObserver(self)
}
```

好了，运行测试.

你发现你可以在后台模式做些你想做的事情了。

在后台模式的最后一个模式中，我们将讨论:`Background Fetching`

## Background Fetch 后台获取

后台获取模式在iOS7的时候已经引进了，它会让你的App在规定的时间出现，并且用最少的电量去呈现最新的信息。

假如你想在你的App中实现一个新的请求，以前的做法是,你可能会用在` viewWillAppear(_:).`中获取新数据。

这个解决方案的问题在于，你的用户看着老数据几分钟之后，这是新数据请求到之后就会覆盖老数据。这会让用户产生疑惑。如果当用户一打开App的时候就是新数据，这样不是更好的用户体验吗？

这就是为什么我们需要后台获取数据的模式。


系统用户使用模式去决定什么时候是最好的实际去进行后台请求，例如，假如你的用户在每天早上的9点打开App,那么你的后台任务应该在之前去请求数据，系统将会决定最好的时机去决定请求数据

为了实现后台请求数据，这里有三件事需要做:

1. 在你的xcode的工程中，打开`Capabilities `选项，检查是否选择了`Background fetch`
2. 使用`setMinimumBackgroundFetchInterval(_:) `去设置适当的请求时间
3. 在你的app的代理方法中实现`application(_:performFetchWithCompletionHandler:`


正如方法名字所示，后台模式通常包含请求信息从一个外部的来源中，比如网络服务。

因为后台模式的这个目的，在当前时间你不会进行网络请求，这个简单将会让你理解每个请求都是在后台模式下完成，而且完全不用担心用外部的服务去测试它。


为了和上个后台模式（一定请求时间的后台任务）做比较，当你进行后台请求数据的时候你仅仅有几分钟的操作-你有最大30秒的时间，如果时间更短则更好。


如果你需要通过这个模式下载比较大的资源文件，那么你需要使用`NSURLSession‘s `后台传输服务

好了，打开`FetchViewController.swift `。

`fetch(_:)`方法将会从一些外部服务中获取一些数据，（这些数据可能是json或者XML）.这可能需要几秒去请求和解析数据，当这个进程是完毕的时候，你的回调函数开始执行。稍后你将会知道为什么这是重要的。

`updateUI()`将会显示格式化后的时间，这个`guard `声明江淮确保`updateLabel`不为nil,确实被加载了。

`time`是一个可选类型，所以它开始不用设置值，刚开始显示的信息为:`Not updated yet`


当视图第一次被加载的时候，这个时候你还没有进行后台请求数据。你直接去调用`updateUI() `，这个时候label显示`Not yet updated`.当更新按钮被点击的时候，它会在执行一个请求，请求完成时候更新UI

编译运行后的效果如下:
![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-FetchNotUpdated.png)


然而，这个时候，后台请求数据模式还不能用

在你的app的`Capabilities `tab页中，第一步就是确保你的后台模式`background fetching`是处于选中状态，如图:

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-EnableBackgroundFetch.png)

现在，打开`AppDelegate.swift`，在`application(_:didFinishLaunchingWithOptions:):`中添加如下代码:
```
UIApplication.shared.setMinimumBackgroundFetchInterval(UIApplicationBackgroundFetchIntervalMinimum)

```

这个后台请求要求设置最小的间隔调用时间，默认的间隔时间是`UIApplicationBackgroundFetchIntervalNever `
比如：当你用户退出或者不想去更新数据了，你也可以设置一个指定的间隔时间。那么系统就会一直处于等待状态直到到达你设定的规定时间。

当心不要设置这个间隔时间太短了，因为频繁的请求可能会非常耗电。

在快到达到你设置的时间之前，系统一直处于等待状态。一般来说，`UIApplicationBackgroundFetchIntervalMinimum `默认属性值就可以了。

最后，为了确保后台请求能够成功你必须实现`application(_:performFetchWithCompletionHandler:).`,添加如下代码:

```
// Support for background fetch
func application(_ application: UIApplication, performFetchWithCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
  if let tabBarController = window?.rootViewController as? UITabBarController,
         let viewControllers = tabBarController.viewControllers
  {
    for viewController in viewControllers {
      if let fetchViewController = viewController as? FetchViewController {
        fetchViewController.fetch {
          fetchViewController.updateUI()
          completionHandler(.newData)
        }
      }
    }
  }
}
```


首先，你需要获得一个`FetchViewContoller `,下一步，转换`rootViewController `为`UITabBarController `,并不是每一个RootViewController都是`UITabBarController `,所以这里用`?`。

但是在我们这个App中，这样转换从不会失败。


下一步，你循环这个tab bar controller中所有的ViewController,最后找到`FetchViewController `,这个App中，你知道最后一个就是FetchViewController，你可以硬编码。但是通过循环的话，会是你的代码更加强健。万一你哪一天想要删除或者添加某个controller了

当完成的时候，你调用`fetch(_:)`方法，你更新UI更新调用`completionHandler `闭包，传递一个参数`. newData `,在操作的最后调用这个闭包，这步操作是很重要的。你指定了在请求数据的过程中会发生什么 做为第一个参数。可能的值为`.newData`,`. noData ` ,`.failed `

为了简单起见，这个教程总是指定`.newData`，这样每次请求都不会失败，每次请求都会由不同的数据。

iOS能够使用这个值让后台请求模式变得更好理解。系统知道这这个关键点去留下一个App快照,这样就能在App切换开关的时候以卡片的形式展现出来。

## 测试后台请求数据模型

后台请求数据的测试方式之一就是做下来等待直到系统决定去做调用。这样很浪费时间。


幸运的是，xcode有一种方式去模拟后台请求，这里有两种情景你需要去测试，一种是当你的App处于后台模式的时候，另外一种就是当你的App刚刚激活的时候。第一种方式是很容易的，仅仅选一下菜单就可以

* 在一个真机上运行
* 如果是模拟器，找到 `Fetch tab`
* 注意此时的消息是"Not yet updated"
* 在xcode的`Debug`菜单中，选择`Simulate Background Fetch`


![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-SimulateBGFetch.png)

* 这个App将会进入后台模式，xcode进入调试模式，

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-ContinueDebugger.png)

* 然后回到这个App上
* 注意这个时间已经更新了。

其它的方式测试就是从一个挂起状态到恢复状态。这里有个可选项让你加载你的app直接进入挂起状态。你想测试这个半成品最好使用一个新的scheme.Xcode配置它很容易。

首先选择`Manage Schemes`菜单:

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-SelectScheme.png)



下一步，选择一个scheme,然后设置中的菜单选择`Duplicate`

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-DuplicateScheme.png)

最后，重命名你的scheme的名称，比如`Background Fetch`,选择` Launch due to a background fetch event`：

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/BM-ConfigureNewScheme.png)

使用这个scheme运行你的App,你将会注意到这个App不会打开，但是一直是挂起状态，现在手动加载它进入到`Fetch`tab页，你将会看到已经执行了更新操作。页面上不再是`Not yet updated`

使用后台请求数据的方式会让你的用户毫无感觉的获取最新的内容。


## 下载完整工程
[下载完整工程](https://koenig-media.raywenderlich.com/uploads/2016/09/TheBackgrounder-Final.zip)

如果你想阅读苹果的官方文档关于后台模式的，请打开[Background Execution](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW1).这个文档解释了所有的后台运行模式。

其中有趣的一个篇章是讨论[being a responsible background app](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW8)，这个里面是一些详情，当你释放你的App在后台运行的时候，你应该知道哪些详情是否和你的App相关。


最后，如果你计划在后台模式下通过网络传输比较大的文件，请学习[NSURLSession](https://www.raywenderlich.com/110458/nsurlsession-tutorial-getting-started)

thanks!