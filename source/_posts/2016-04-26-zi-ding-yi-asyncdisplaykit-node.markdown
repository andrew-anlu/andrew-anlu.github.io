---
layout: post
title: "自定义AsyncDisplayKit-Node"
date: 2016-04-26 15:13:13 +0800
comments: true
categories: swift
---


![node](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_NodeHierarchy.png)

假设你已经读过[AsyncDisplayKit入门](http://andrew-anlu.github.io/blog/2016/04/15/asyndisplaykitru-men-pian/),下面我们将继续介绍AsyncDisplayKit.

<!--more-->

这篇教程将解释如何充分利用框架探索AsyncDisplayKit节点的层次结构，通过这样做，你将会更加熟悉AsyncDisplayKit这个有名的框架，并且会使你的app程序异常平滑，同时能够构建灵活的和可重用的ui

这AsyncDisplayKit中一个核心概念就是`node`,正如你所知，AsyncDisplayKit nodes 是一个线程安全的抽象于UIview之上的，（UIview不是线程安全的）,你可以学习更多AsyncDisplayKit在[AsyncDisplayKit’s Quick Start introduction](http://asyncdisplaykit.org/)


好消息是你已经知道了UIKit,那么你已经了解AsyncDisplayKit一些属性和方法了，因为API是很相似的。随后，你将会学到:

* 如何去创建你自己的 AsDisplayNode
* 如何嵌套一个node到一个单个Node容器中，并且管理和重用
* 如何用一个Node层次结构支持View的层次结构，因为你自动减少在主线程绘制的机会，保持你的界面平滑和响应

下面是你将要做的：你将要创建一个容器Node,并且添加两个子Node-其中一个是图片node,其中一个是文本node,你将会看到容器是如何测量他们的子Node尺寸和大小的.最后，你将会转化一个已经存在的UIview容器为一个ASDisplaytNode子类。

效果图如下:

![node](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_NodeHierarchies-7-e1434332214236.jpg)


##开始
这个app,你将要创建一个卡片展示世界上一个奇观-泰姬陵

下载[开始工程](http://www.raywenderlich.com/wp-content/uploads/2015/06/Wonders-Starter.zip)

这个工程只有一个ViewController,这个工程是用 cocoaPods构建的，所以你必须安装 pods,然后创建 Podfile,引入 AsyncDisplayKit.

>*注意*
>如果你不了解 Pods,请学习[Introduction to CocoaPods Tutorial](http://www.raywenderlich.com/64546/introduction-to-cocoapods-2)
>


打开  ViewController.swift，然后注意到这个view controller只有一个常量`card`,它保存了一个泰姬陵的模型对象，你将会使用这个模型创建一个卡片的node去展示。

编译运行工程，你将会看到一个空的黑色屏幕
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_NodeHierarchies-1-281x500.jpg)

##创建并且显示一个容器Node
现在你要开始构建你的第一个Node层次结构，它是非常相似的和创建一个UIview的层次结构

打开 Wonders-Bridging-Header.h，然后添加如下代码:

```
#import <AsyncDisplayKit/ASDisplayNode+Subclasses.h>
```

`ASDisplayNode+Subclasses.h`是AsDisplayNode的一个内部方法，你需要在`ASDisplayNode`子类中实现这个header中定义的方法，但是你只能调用这些方法在你的 ASDisplayNode的子类中，这是和` UIGestureRecognizer`模式很相似的。

打开CardNode.swift，然后添加ASDisplayNode子类实现:

```
class CardNode: ASDisplayNode {}
```

定义了一个新的ASDisplayNode子类，你将会把它作为一个容器去处理用户的交互

打开ViewController.swift,在viewDidLoad()中实现如下:


```
override func viewDidLoad() {
  super.viewDidLoad()
  // Create, configure, and lay out container node
  let cardNode = CardNode()
  cardNode.backgroundColor = UIColor(white: 1.0, alpha: 0.27)
  let origin = CGPointZero
  let size = CGSize(width: 100, height: 100)
  cardNode.frame = CGRect(origin: origin, size: size)
  // Create container node’s view and add to view hierarchy
  view.addSubview(cardNode.view)
}
```
上面的代码创建了一个新的卡片node,它定位在左上角并且高宽都是100.
不用担心之前的约束，你将会定位到中心在这个view controller中。

编译运行 ：

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_NodeHierarchies-2-281x500.jpg)

太好了！你已经有了一个自定义的node在屏幕上显示，下一步是给你的子node取名为 `CardNode`,并且计算它的尺寸。在view中居中这是必须要知道的，你应该理解node布局引擎的工作原理。


##Node布局引擎
下一步是询问Node去计算自己的尺寸通过调用`measure(constrainedSize:)`,你将会通过方法中constrainedSize参数去告诉node去计算一个尺寸去适应constrainedSize

通俗的说,这意味着计算大小不能大于限制大小。例如,考虑下面的图
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_measure.png)

这个展示了一个约束的尺寸使用确定的宽度和高度，这个计算尺寸是和宽度相等的，但是比高度要小，它可能和宽度和高度都相等，或者比宽度和高度都要小，但是宽度和高度都不允许比约束的尺寸要大。

这个工作和UIView's`sizeThatFits(size:)`方法很相似。但是不同之处在于`measure(constrainedSize:)`计算它的尺寸，允许你访问缓存
node的calculatedSize属性。

一个例子就是当计算的高度和宽度的尺寸比约束尺寸更小的时候：
![lgo](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_constrained2.png)

这里图片的尺寸是更小比约束的尺寸，没有任何的sizing-to-fit，这里计算的尺寸比约束尺寸更小。

原因就是AsyncDisplayKit经常需要花费时间去计算尺寸，读取一个图片从硬盘中，可能会非常慢，通过合并尺寸到nodeApi,这个是线程安全的操作，这意味着计算尺寸可以在后台线程中运行！优雅的！它是一个有用的特性让你的UI程序平滑如黄油一般。

一个Node将会运行计算尺寸加入它没有存储值的话，假如这个约束尺寸提供是不同的，约束尺寸常常决定缓存的计算大小。


在程序中，工作如下:

* measure(constrainedSize:)返回一个存储你的尺寸或者进行一个计算尺寸通过调用calculateSizeThatFits(constrainedSize:)
* 你替换所有的尺寸计算通过`calculateSizeThatFits(constrainedSize:) `在你的ASDisplayNode子类中


>*注意*
>calculateSizeThatFits(constrainedSize:)是ASDisplayNode的内部方法，你不应该在外部调用它
>


##测量Node的大小

现在是时候测量你的自己的node大小了。

打开CardNode.swift,然后替换这个类中的代码:

```
class CardNode: ASDisplayNode {
 
  override func calculateSizeThatFits(constrainedSize: CGSize) -> CGSize {
    return CGSize(width: constrainedSize.width * 0.2, height: constrainedSize.height * 0.2)
  }
 
}
```

到现在为止，这个方法返回的大小是约束提供尺寸的20%.

打开ViewController.swift,删除viewDidLoad() 中的实现，然后实现下面的`createCardNode(containerRect:) `方法:

```
/* Delete this method
 
override func viewDidLoad() {
  super.viewDidLoad()
  // 1
  let cardNode = CardNode()
  cardNode.backgroundColor = UIColor(white: 1.0, alpha: 0.27)
  let origin = CGPointZero
  let size = CGSize(width: 100, height: 100)
  cardNode.frame = CGRect(origin: origin, size: size)
 
  // 2
  view.addSubview(cardNode.view)
}
*/
 
func createCardNode(#containerRect: CGRect) -> CardNode {
  // 3
  let cardNode = CardNode()
  cardNode.backgroundColor = UIColor(white: 1.0, alpha: 0.27)
  cardNode.measure(containerRect.size)
 
  // 4
  let size = cardNode.calculatedSize
  let origin = containerRect.originForCenteredRectWithSize(size)
  cardNode.frame = CGRect(origin: origin, size: size)
  return cardNode
}
```
上面的代码做的工作如下:

1. 删除之前创建的代码，配置和布局代码
2. 删除之前的代码，把node加入到view的层次结构中
3. createCardNode(containerRect:) 创建了一个卡片Node使用相同的背景颜色和容器node,它用作一个容器去约束卡片node的大小，所以这个卡片Node不能比containerRect’的尺寸更大
4. 通过originForCenteredRectWithSize(size:) 方法设置卡片到中心位置。


定位到createCardNode(containerRect:) 这个方法，替换viewDidLoad():

```
override func viewDidLoad() {
  super.viewDidLoad()
  let cardNode = createCardNode(containerRect: UIScreen.mainScreen().bounds)
  view.addSubview(cardNode.view)
}
```

当视图加载的时候，createCardNode(containerRect:)创建一个新的CardNode,这个卡片的Node不能比主屏幕的bounds尺寸更大。

在这个观点中，这个ViewController的视图还没有加载出来，因此，它不是安全的对用于ViewContrller的bounds尺寸。所以你使用主屏幕的尺寸去约束卡片Node的大小。


运行起来，尽管达不到优雅，因为这个视图控制器横跨了整个屏幕，稍后，我们将添加适当的方法，现在，它工作还可以。


编译运行，你将会看到你的node在正中心了.
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_NodeHierarchies-3-281x500.jpg)

##异步节点设置和布局
有时它会花费人们很多时间去解析复杂的层次结构，假如这是在主线程发生，这将会阻塞主线程，假如你想取悦用户的话，你不能让用户一直等待，之前用户是不能和程序有任何交互的。

因为这个原因，你将会在后台线程创建设置nodes，这样你能避免阻塞主线程。


在createCardNode(containerRect:)和viewDidLoad():中实现addCardViewAsynchronously(containerRect:)


```
func addCardViewAsynchronously(#containerRect: CGRect) {
  dispatch_async(dispatch_get_global_queue(QOS_CLASS_BACKGROUND, 0)) {
    let cardNode = self.createCardNode(containerRect: containerRect)
    dispatch_async(dispatch_get_main_queue()) {
      self.view.addSubview(cardNode.view)
    }
  }
}
```


`addCardViewAsynchronously(containerRect:) `创建这个`CardNode`在一个线程队列中，这是很好的，因为nodes是线程安全的！创建好之后，配置这个Node,然后异步调用主线程把Node添加到视图控制器中--毕竟，UIKit不是线程安全的！

>*注意*
>一旦你创建了Node的视图，所有访问node的节点只在主线程
>


重新实现viewDidLoad()通过调用` addCardViewAsynchronously(containerRect:):`

```
override func viewDidLoad() {
  super.viewDidLoad()
  addCardViewAsynchronously(containerRect: UIScreen.mainScreen().bounds)
}
```

这将不再阻塞主线程，确保用户界面是可以随时响应的。

编译运行，和之前一样，不过所有的操作都是在后台线程执行的

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_NodeHierarchies-3-281x500.jpg)

##约束节点大小

记得之前我你会使用一个更优雅的解决方案节点不仅仅依靠屏幕尺寸的大小
?现在我来兑换我的诺言

打开 ViewController.swift，添加下面的属性：

```
var cardViewSetupStarted = false
```
然后替换viewDidLoad()为viewWillLayoutSubviews():


```
/* Delete this method
override func viewDidLoad() {
  super.viewDidLoad()
  addCardViewAsynchronously(containerRect: UIScreen.mainScreen().bounds)
}
*/
 
override func viewWillLayoutSubviews() {
  super.viewWillLayoutSubviews()
  if !cardViewSetupStarted {
    addCardViewAsynchronously(containerRect: view.bounds)
    cardViewSetupStarted = true
  }
}
```

替换掉了住屏幕的尺寸，这上面的逻辑是视图控制器的尺寸去约束卡片的尺寸。


现在它是线程安全的,去用这个视图控制器的尺寸，当viewWillLayoutSubviews()替换调viewDidLoad()。这一次在这个声明周期中，这个视图控制器已经设置好了它的尺寸。

这种加载方式是很出众的，因为一个视图控制器能被设置任何尺寸,你不想依赖视图控制器去横跨整个屏幕

![log](http://www.raywenderlich.com/wp-content/uploads/2015/06/screen_size_tyranny.png)


这个视图可能会加载多次，所以viewWillLayoutSubviews()能被调用多次，你仅仅想创建这个CardNode一次，所以这就是为什么你需要一个cardViewSetupStarted标识去证明这个视图控制创建多次。

编译运行：

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_NodeHierarchies-3-281x500.jpg)


##Node层次结构
现在你有一个空的Node容器在屏幕上，现在你想展示一些内容，方式的就是你添加一些子node到这个Node容器中，下面的结构图展示了简单的Node结构图

![node](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_Nodes.png)


添加子Node看起来将会非常像你在UIview中添加子View的过程。

第一步你需要添加一个图片的node,但是首先你的需要了解容器Node是如何布局子Node的。

##子Node的布局

你现在知道了如何去测量容器的尺寸和如何去计算容器尺寸去布局容器内的node视图，但是这个容器node是如何布局它们的子Node呢？

有以下两步:

1. 第一，你测量每个子Node的尺寸在`calculateSizeThatFits(constrainedSize:). `中，这将确保每一个子Node都缓存一个计算好的尺寸
2. 在UIKIt的主线程布局中，AsyncDisplayKit将会调用`layout()`在你的自定义的ASDisplayNode子类中，`layout()`的工作就像UIview的 `layoutSubivews()`，除了这个`layout()`不会计算所有子视图的尺寸，布局()简单查询计算每个节点的大小

回到UI上，这个泰姬陵卡片尺寸将会和它的图片大小一样，并且这个标题将会适合它的大小。最简单的方式就是去测量这个`泰姬陵图片`node的大小并且使用这个结果去约束文本node的尺寸，所以这个文本node将会适合图片尺寸的大小


接下来，你将要使用lay out去布局你的卡片子Node,让我们开始

##添加一个子Node

打开CardNode.swift,然后添加下面的代码在CardNode

```
// 1
let imageNode: ASImageNode
 
// 2
init(card: Card) {
  imageNode = ASImageNode()
  super.init()
  setUpSubnodesWithCard(card)
  buildSubnodeHierarchy()
}
 
// 3
func setUpSubnodesWithCard(card: Card) {
  // Set up image node
  imageNode.image = card.image
}
 
// 4
func buildSubnodeHierarchy() {
  addSubnode(imageNode)
}
```

上面的代码做的工作如下:

1. 图片Node属性:这行代码添加一个图片Node的属性，去引用卡片图像的子Node
2. 初始化:这个设计的初始化使用一个卡片模型保存卡片的图像和文本
3. 设置子Node:这个方法使用卡片模型的图像去保存开始项目中的子Node图片
4. 容器层次结构:你设置Node的层次结构就像你能设置视图的层次结构一样，这个方法创建卡片的层次结构通过添加一个子Node来实现。

下一步，实现calculateSizeThatFits(constrainedSize:):

```
override func calculateSizeThatFits(constrainedSize: CGSize) -> CGSize {
  // 1 
  imageNode.measure(constrainedSize)
 
  // 2 
  let cardSize = imageNode.calculatedSize
 
  // 3 
  return cardSize
}
```

上面的代码做的工作如下:

1. 这个卡片的尺寸就会匹配背景图片的大小，这个测量背景图片的尺寸会适合内嵌的约束尺寸，所有Node的子类都会被AsyncDisplay框架感知并且知道如何设置它们的尺寸，包括AsImageNode
2. 这行代码临时存储图片计算的尺寸，特别说明的是，它使用图片Node测量尺寸正如卡片Node尺寸去约束子NOde，当添加更多的子Node的时候，你将会使用这个值
3. 返回这个卡片Node的计算好的尺寸

下一步，覆盖layout():

```
override func layout() {
  imageNode.frame =
    CGRect(origin: CGPointZero, size: imageNode.calculatedSize).integerRect
}
```

这个图片的逻辑位置在左上角，坐标轴原点，它也确保了这个图片Node的frame没有任何多余的值，所以你能避免像素边界显示问题

注意这个方法是如何使用图片的Node去缓存计算尺寸的在布局的过程中。

因为这个图片Node的尺寸决定了卡片Node的尺寸，这个图片将会横跨这个卡片

回到ViewController.swift，然后添加一个createCardNode(containerRect:), 替换到之前初始化的`CardNode `：

```
let cardNode = CardNode(card: card)
```
这行代码使用新的初始化去添加一个卡片Node,这个卡片值将会在初始化的时候被传入，并且存储这个泰姬陵的卡片模型

编译运行:
![t](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_NodeHierarchies-6-281x500.jpg)

漂亮，你成功的创建了容器Node并且呈现了Node的层次结构，当然，这个很简单，但是它是一个完成的Node层次！

##添加更多的子Node

也许你已经注意到了，没有添加标题，不用担心，让我们开始添加吧。

打开CardNode.swift，然后添加下面的属性titleTextNode在类中：

```
let titleTextNode: ASTextNode
```
在`init(card:)`方法中初始化`titleTextNode `，在super.init()这行代码之前:

```
titleTextNode = ASTextNode()
```
添加下面的这行代码在`setUpSubnodesWithCard(card:):`

```
titleTextNode.attributedString = NSAttributedString.attributedStringForTitleText(card.name)

```
这行代码给titleTextNode设置了富文本内容，`attributedStringForTitleText(text:) `是一个帮助方法被加到了NSAttributedString扩展属性中。在之前的开始工程中，它创建了富文本使用提供的标题。

下一步，在`buildSubnodeHierarchy():`的结尾增加如下代码:

```
addSubnode(titleTextNode)
```

要确保它是在添加到图片Node的后面，不然图片将会遮盖住标题。

在内置的方法`calculateSizeThatFits(constrainedSize:)`中，添加如下代码:

```
titleTextNode.measure(cardSize)
```
使用子Node的测量尺寸来作为卡片的约束尺寸


添加如下的布局代码layout():

```
titleTextNode.frame =
  FrameCalculator.titleFrameForSize(titleTextNode.calculatedSize, containerFrame: imageNode.frame)

```

这行代码计算标题Node的frame通过工具方法FrameCalculator

编译运行:
![test](http://7xkxhx.com1.z0.glb.clouddn.com/ASDK_NodeHierarchies-7-281x500.jpg)


至此，你已经构建了完整的子Node层次结构，使用了容器Node,并且添加了两个子Node

##完整工程

[完整工程在这里下载](https://github.com/TLSwiftDemo/Wonders/archive/master.zip)


have fun 😀~
