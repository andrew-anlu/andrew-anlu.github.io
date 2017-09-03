---
layout: post
title: "AsynDisplayKit入门篇"
date: 2016-04-15 10:22:30 +0800
comments: true
categories: Swift
---

##前言
FaceBook的Paper团队给我们开源了一个很棒的库:[AsynDisplayKit](https://github.com/facebook/AsyncDisplayKit),这个库能让你通过将图像解码，布局以及渲染操作都放到后台线程处理，从而带来了快速响应的用户界面，也就是说不再会因为界面卡顿尔阻断用户交互。

<!--more-->


例如，对于非常复杂的界面，你可以使用 AsyncDisplayKit构建它而得到一种如丝般顺滑的，60帧每秒的滑动体验。而平常的UIkit优化就不太可能克服这样的性能挑战。

从本教程中，你将从一个初始项目开始，它主要有一个UICollectionView的滑动问题，而使用AysncDisplayKit将大大提供其滑动性能。一路上，你将学会如何在旧项目上使用AsyncDisplaykit.

>*注意*，在本教程之前，你应该熟悉 Swift,Core Animation以及Core Graphics.
>


##开始
开始之前，请先看看[AsyncDisplayKit介绍](https://code.facebook.com/posts/721586784561674/introducing-asyncdisplaykit-for-smooth-and-responsive-apps-on-ios/),对它有个简要的概念，知道它是解决什么问题的。


准备好了，就下载[开始项目](http://www.raywenderlich.com/wp-content/uploads/2015/04/Layers-Start.zip),你需要至少Xcode6.1和iosSDK 8.1来编译它，如果用最新Xcode打开，swift语法需要做下转换和修改，请自行解决这些兼容问题.


你要研究的项目是由UICollectionView制作的卡片式界面来描述不同的雨林动物，每张信息卡上包含了一个图片，名字以及一个队雨林动物的描述，卡片的背景图是主图片的模糊感，视觉上设计的效果保证了文字的清晰可读。

![logo](http://cc.cocimg.com/api/uploads/20141124/1416797117122129.png)
在Xcode中，打开初始项目的 Layers.xcworkspace。

在本教程里，请遵循以下原则以体会AsyncDisplayKit的那些十分吸引人的好处。

将应用运行到真机上，在模拟器上很难看出性能改善。

应用是通用的，但在ipad上看起来最好。

最后，要真正感谢这个库为你所有的事情，请尽量在最旧的ios8.1的设备上去运行该应用，第三代的ipad最好，因为它随让是视网膜屏，但是运行的不是很快。

运行该项目，效果如下:

![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0002.png)

试着滑动CollectionVie并注意那可怜的帧率，在第三代ipad上，帧率只有15-20FPS，实在丢掉太多帧了，在本教程的最后，你能在60 FPS的帧率下滑动它.

>*注意*
>你所看到的图像都是在App的asset目录里，并不是从网络里获取的。
>
>


在一个旧项目中使用AsyncDisplayKit前，你应该通过Instruments测量你的UI的性能，这样才能有一个基准线以便对比改动的效果。

最重要的是，你要知道是CPU-绑定，还是GPU-绑定，也就是说是CPU还是GPU拉低了应用的帧率，这个信息会告诉你该充分利用AsyncDisplayKit的那个特性以优化应用的性能。

如果你有时间，看看之前提到的WWDC2012 session 或在真实的设备上使用Instruments来评估初始项目的时间曲线。滑动性能是CPU-绑定的，你能猜到是什么原因导致了 CollectionView 丢掉了这么多帧吗?


##为项目准备好使用 AsyncDisplayKit
在旧项目里使用AsyncDisplayKit，总结起来就是使用 Display Node 层级结构替换视图层级结构或Layer树，各种Display Node是AsyncDisplayKit的关键所在，它们位于视图之上，而且是线程安全的，也就是说之前在主线程才能执行的任务现在也可以在非主线程中执行。这就是减少主线程的工作量以执行其它操作，例如处理触摸事件，或如下本应用的情况下，处理CollectionView的滑动。

这就意味着在本教程里，你的第一步是移除视图层级结构。


###移除视图的层次结构
打开 RainForestCardCell.Swift并删除awakeFromNib() 中所有的 addSubview(....)调用，然后得到如下:

```
override func awakeFromNib() {
  super.awakeFromNib()
  contentView.layer.borderColor =
    UIColor(hue: 0, saturation: 0, brightness: 0.85, alpha: 0.2).CGColor
  contentView.layer.borderWidth = 1
}
```

接下来，替换LayoutSubviews()的内容如下:

```
override func layoutSubviews() {
  super.layoutSubviews()
}
```

再讲configureCellDisplayWithCardInfo(cardInfo:)的内容替换如下:

```
func configureCellDisplayWithCardInfo(cardInfo: RainforestCardInfo) {
  //MARK: Image Size Section
  let image = UIImage(named: cardInfo.imageName)!
  featureImageSizeOptional = image.size
}
```

删除RainforestCardCell的所有视图属性，只留一个如下:

```
class RainforestCardCell: UICollectionViewCell {
  var featureImageSizeOptional: CGSize?
  ...
}
```

最后编译运行，你看的的是黑洞洞的一片:

![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0001.jpg)

现在所有的cell都空了，滑动起来就很顺滑，你的目标是保证之后添加完各个Node之后，依然顺滑如初。


###添加一个占位图
打开RainforestCardCell.swift,给RainforestCardCell添加一个可选的 CALayer变量，名为placeholderLayer：

```
class RainforestCardCell: UICollectionViewCell {
  var featureImageSizeOptional: CGSize?
  var placeholderLayer: CALayer!
  ...
}
```

你之所以需要一个占位图是因为显示会异步完成，如果这个过程需要一些时间，那用户就会看到空的cell 。就如同如果你要从网络上获取图像，那么就需要用占位图来填充Cell,这能让你的用户知道内容还没准备好。随让在我们这种情况下，你是在后台线程绘制而不是从网络上下载。


在awakeFromNib()里，删除contentView的border设置，再创建并配置一个placeholderLayer.将其添加到cell的contentview的layer上，现在这个方法如下:

```
override func awakeFromNib() {
  super.awakeFromNib()
 
  placeholderLayer = CALayer()
  placeholderLayer.contents = UIImage(named: "cardPlaceholder")!.CGImage
  placeholderLayer.contentsGravity = kCAGravityCenter
  placeholderLayer.contentsScale = UIScreen.mainScreen().scale
  placeholderLayer.backgroundColor = UIColor(hue: 0, saturation: 0, brightness: 0.85, alpha: 1).CGColor
  contentView.layer.addSublayer(placeholderLayer)
}
```

在layoutSubviews()里，你需要布局placeholderLayer.替换这个方法为:

```
override func layoutSubviews() {
  super.layoutSubviews()
 
  placeholderLayer?.frame = bounds
}
```
编译并运行，你从虚无的边缘回来了:
![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0003.png)

普通的CALayer不是由UIview支持的，当它们改变frame时，默认会有隐式动画，这就是你看到layer在布局放大时，要修复这个问题，改动layoutSubviews()如下:

```
override func layoutSubviews() {
  super.layoutSubviews()
 
  CATransaction.begin()
  CATransaction.setValue(kCFBooleanTrue, forKey: kCATransactionDisableActions)
  placeholderLayer?.frame = bounds
  CATransaction.commit()
}
```

编译运行，问题解决了。

现在占位图不会乱动，不再动画它们的frame了。

##第一个Node
重建App的第一步就是给每一个UICollectionView cell 添加一个背景图片的Node,步骤如下:

1. 创建，布局并添加一个图像Node到UICollectionView cell
2. 处理cell重用Node和它们的layer
3. 模糊图像Node


但在做之前，打开 Layers-Bridging-Header.h 并导入 AsyncDisplayKit :

```
#import <AsyncDisplayKit/AsyncDisplayKit.h>
#import <AsyncDisplayKit/_ASDisplayLayer.h>
```
这会让所有的Swift文件都能访问AsyncDisplayKit类库。

编译一下，确保没有错误

现在，我们来看看Collectin View的祖成:

* View Controller: RainforestViewController没有什么花哨的东西，它只是为所有的雨林卡片获取一个数据数组，并为UIcollectioNView实现DataSource.事实上，你不需要花太多时间在这个上
* DataSource:大部分时间都将花在Cell类的RainforestCardCell上，ViewController出队每个cell，并将雨林卡片的数据用configureCellDisplayWithCardInfo(cardInfo:) 传给它。cell就使用这个数据来配置自身.
* Cell: 在configureCellDisplayWithCardInfo(cardInfo:)里，cell创建，配置，布局以及添加Node到它自己身上。这就意味着每次ViewController出队一个cell,这个cell就会创建并添加它自己一个新的Node层级结构

如果你使用View而不是Node，那么这样做对于性能来说就不是最佳策略。但因为你可以异步的创建，配置以及布局，而且Node也是异步地绘制，所以这不会是一个问题。真正的难点是在cell准备重用时取消任何在进行额异步操作并移除旧的node.


然而，在实际生产中，你最好使用ASRangeController来缓存你的Node,这样你就不用每次在cell重用时重建它的Node层级结构，ASRangeController超出了本教程的范围。

OK，动手!

###添加背景图片Node

现在你要走一遍用Node配置cell的过程，一次一步：
打开RainforestCardCell.swift 并替换configureCellDisplayWithCardInfo(cardInfo:) 为:

```
func configureCellDisplayWithCardInfo(cardInfo: RainforestCardInfo) {
  //MARK: Image Size Section
  let image = UIImage(named: cardInfo.imageName)!
  featureImageSizeOptional = image.size
 
  //MARK: Node Creation Section
  let backgroundImageNode = ASImageNode()
  backgroundImageNode.image = image
  backgroundImageNode.contentMode = .ScaleAspectFill
}
```
这就创建并配置了一个ASImageNode常量，叫做backgroundImageNode.

AsyncDisplayKit带有好几种Node类型，包括ASImageNode，用于显示图片。它相当于UIImageView,除了ASImageNode是默认异步的解码图片。

添加如下代码到configureCellDisplayWithCardInfo(cardInfo:) 底部:

```
backgroundImageNode.layerBacked = true
```
这让backgroundImageNode变为layer支持的Node.

Node可由UIview支持或CALayer支持，当node需要处理事件时（例如触摸事件），你就要使用UIView支持的Node.如果你不需要处理事件，只需要显示一下内容，那使用Layer支持的Node会更加轻量，因此可以获得一个小的性能提升。

因为本教程的APP不需要处理事件，所以你可以让所有的Node都设置为Layer支持的。在上面的代码中，由于backgroundImageNode为Layer支持的。AsyncDisplayKit会创建一个CALayer用于雨林动物图像内容的显示.


继续在 configureCellDisplayWithCardInfo(cardInfo:)添加如下代码:

```
//MARK: Node Layout Section
backgroundImageNode.frame = FrameCalculator.frameForContainer(featureImageSize: image.size)
```
这里使用FrameCalculator为backgroundImageNode布局.

FrameCalculator是一个帮助类，负责给每个Node布局。注意所有的东西都是手动布局的，没有使用AutoLayout约束。**如果你需要构建自适应布局或者本地化驱动的布局，那就要注意，因为你不能给Node添加约束**

接下来，添加如下代码到configureCellDisplayWithCardInfo(cardInfo:)底部:

```
//MARK: Node Layer and Wrap Up Section
self.contentView.layer.addSublayer(backgroundImageNode.layer)
```
这句将backgroundImageNode的layer添加到cell ContentView的layer上。

>**注意**
>AsyncDisplayKit会为backgroundImageNode创建一个Layer.然而，你必须要将Node放到某个Layer树中才能在屏幕上显示，这个Node会异步地绘制，所以直到绘制完成，它的内容都不会显示，尽管它的Layer已经在一个Layer树中。
>
>


从技术的角度来说，layer一直都存在。但渲染图像是异步进行的。layer初始化时没有内容，一旦渲染完成，layer的contents就会更新为包含图像的内容。

在这个点，cell的contentView的layer将会包含两个Sublayer:一个占位图和Node的layer。在node完成绘制前，只有占位图会显示。


注意到configureCellDisplayWithCardInfo(cardInfo:)会在每次cell出队时被调用。每次cell被回收，这个逻辑会添加一个新的sublayer到cell的contentview layer上。不要担心，你很快会解决这个问题。

回到RainforestCardCell.swift开头，给RainforestCardCell添加一个ASImageNode变量存为属性backgroundImageNode，如下:

```
class RainforestCardCell: UICollectionViewCell {
  var featureImageSizeOptional: CGSize?
  var placeholderLayer: CALayer!
  var backgroundImageNode: ASImageNode? ///< ADD THIS LINE
  ...
}
```

你之所以需要这个属性是因为必须要有某个东西将backgroundImageNode的引用保留住，否则ARC就会将其释放，也就不会有任何东西显示出来了--即使Node的在一个layer树中，你依然需要保留Node.

在configureCellDisplayWithCardInfo(cardInfo:)底部的Node Layer and Wrap Up Section ,设置cell新的backgroundImageNode为之前的backgroundImageNode：

```
self.backgroundImageNode = backgroundImageNode
```


下面是完整的configureCellDisplayWithCardInfo(cardInfo:) 方法：

```
func configureCellDisplayWithCardInfo(cardInfo: RainforestCardInfo) {
  //MARK: Image Size Section
  let image = UIImage(named: cardInfo.imageName)!
  featureImageSizeOptional = image.size
 
  //MARK: Node Creation Section
  let backgroundImageNode = ASImageNode()
  backgroundImageNode.image = image
  backgroundImageNode.contentMode = .ScaleAspectFill
  backgroundImageNode.layerBacked = true
 
  //MARK: Node Layout Section
  backgroundImageNode.frame = FrameCalculator.frameForContainer(featureImageSize: image.size)
 
  //MARK: Node Layer and Wrap Up Section
  self.contentView.layer.addSublayer(backgroundImageNode.layer)
  self.backgroundImageNode = backgroundImageNode
}
```
编译运行，观察AsyncDisplaytKit是如何异步地使用图像设置Layer的Contents的。这能让你在CPU还在绘制layer的内容的同事上下滑动页面。

![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0006.png)

如果你运行在旧设备上，注意图像是如何弹出到位置--这是爆米花效果，但不总是让人喜欢。本教程的最后一节将会搞定这个不令人愉快的弹出效果，给你展示图像如何弹入弹出，如同摇滚巨星。

如同之前所讨论的，新的Node会在每次cell被重用时创建，这并不理想，因为这意味着新的Layer会在每次cell被重用时加入。

如果你想看看Sublayer堆积太多的影响，那就不停的滑上滑下，然后加断点打印出cell的contentview的Layer的sublayers属性。你会看到很多layer,这不好.

##处理cell重用

继续RainforestCardCell.swift,给RainforestCardCell 添加一个contentLayer的CALayer属性，这个属性也是一个可选类型:

```
class RainforestCardCell: UICollectionViewCell {
  var featureImageSizeOptional: CGSize?
  var placeholderLayer: CALayer!
  var backgroundImageNode: ASImageNode?
  var contentLayer: CALayer? ///< ADD THIS LINE
  ...
}
```

你将使用此属性去移除Cell的ContentView的Layer树中旧的Node Layer.虽然你可以简单地保留Node并访问其Layer属性，但上面的写法更加明确。

添加如下代码到configureCellDisplayWithCardInfo(cardInfo:) 结尾：

```
self.contentLayer = backgroundImageNode.layer
```
这句让backgroundImageNode的Layer保留到contentLayer属性。

替换prepareForReuse()的实现如下:

```
override func prepareForReuse() {
  super.prepareForReuse()
  backgroundImageNode?.preventOrCancelDisplay = true
}
```
因为AsyncDisplaytKit能够异步地绘制Node,所以Node让你能预防从头绘制或取消任何在进行的绘制。无论是你需要预防或取消绘制，都可将preventOrcancelDisplay设置为true,如上面的代码所示，在本来中，你要在cell被重用前取消任何正在进行的绘制活动.

接下来，添加如下代码到prepareForReuse（）尾部:

```
contentLayer?.removeFromSuperlayer()
```
这将contentLayer从其SuperLayer(也就是contentview的Layer)中移除.

每次一个cell被回收时，这个代码就移除Node的旧Layer,因而解决了堆积问题。所以在任何时间，你的Node最多只有两个sublayer:占位图和Node的Layer.

接下来添加如下代码到prepareForReuse()尾部:


```
contentLayer = nil
backgroundImageNode = nil
```
这确保cell释放它们的引用，这样如有必要，ARC才好做清理工作。

编译运行，这次，没有sublayer会堆积的问题，且所有不必要的绘制都将被取消.

![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0006.png)

是时候来点模糊效果了!
![blue](http://www.raywenderlich.com/wp-content/uploads/2014/11/asyncscroll.png)


##模糊图像
要实现模糊图像，你要添加一个额外的步骤到图像Node的显示过程里。

继续RainforestCardCell.swift ,在configureCellDisplayWithCardInfo(cardInfo:) 的设置backgroundImageNode.layerBacked 的后面，添加如下代码:

```
backgroundImageNode.imageModificationBlock = { input in
  if input == nil {
    return input
  }
  if let blurredImage = input.applyBlurWithRadius(
    30,
    tintColor: UIColor(white: 0.5, alpha: 0.3),
    saturationDeltaFactor: 1.8,
    maskImage: nil, 
    didCancel:{ return false }) {
      return blurredImage
  } else {
    return image
  }
}
```

ASImageNode的imageModificationBlock给你一个积水在显示之前去处理底层的图像，这是非常实用的功能，它让你对图像Node做一些操作，例如添加滤镜等。

在上面的代码中，你使用imageModificationBlock来为cell的背景图像应用模糊效果。关键点就是图像Node将会绘制它的内容并在后台执行这个闭包，而主线程依然顺滑流畅。这个闭包接受原始的UIImage并返回一个修改过的UIimage.

上面的代码使用了UIImage的模糊category,它由Apple在WWDC2013提供。使用了Accelerate framework 在CPU上模糊图像。因为模糊会消耗很多时间和内存，这个版本的category被修改为包含了取消机制。这个模糊方法将定期调用didCancel闭包来决定是否应该停止模糊。

现在，上面的代码给didCancel简单地返回false,之后你可以重写didCancel闭包.

>*注意*
>还记得第一次运行APP时collectionView那可怜的滑动效果吗？模糊方法阻塞了主线程。通过使用AsyncDisplayKit将模糊放入后台线程，你就大幅度地提高了CollectionView的滑动性能。简直天壤之别。
>



编译并运行，观察模糊效果:
![lgo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0009.png)
注意你可以非常流畅的滑动页面了

当Collectionview出队一个cell时，一个模糊操作将开始后台线程，当用户快速滑动时，CollectionView会重用每个cell多次，并开始许多模糊操作。我们的目标是在cell准备重用时取消正在进行中的模糊操作。

你已经在prepareForReuse()里取消了Node的绘制操作，但一旦控制被移交给处理你图像修改的闭包，那就是你的责任来处理Node的preventOrCancelDisplay的设置。

##取消模糊操作
要取消进行中的模糊操作，你需要实现模糊方法的didCancel闭包.

添加一个捕获列表到imageModificationBlock以捕捉一个backgroundImageNode的weak引用:

```
backgroundImageNode.imageModificationBlock = { [weak backgroundImageNode] input in
   ...
}
```

你需啊哟weak引用来避免闭包和图像Node之间的循环引用问题。你将使用这个weak  backgroundImageNode 来确定是否要取消模糊操作。

是时候构建模糊取消碧波啊了。添加如下代码到imageModificationBlock:

```
backgroundImageNode.imageModificationBlock = { [weak backgroundImageNode] input in
  if input == nil {
    return input
  }
 
  // ADD FROM HERE...
  let didCancelBlur: () -> Bool = {
    var isCancelled = true
    // 1
    if let strongBackgroundImageNode = backgroundImageNode {
      // 2
      let isCancelledClosure = {
        isCancelled = strongBackgroundImageNode.preventOrCancelDisplay
      }
 
      // 3
      if NSThread.isMainThread() {
        isCancelledClosure()
      } else {
        dispatch_sync(dispatch_get_main_queue(), isCancelledClosure)
      }
    }
    return isCancelled
  }
  // ...TO HERE
 
  ...
}
```

下面解释一下这些代码：

1. 得到backgroundImageNode的Strong引用，准备其干活。如果backgroundImageNode在本次运行时小时，那么isCancelled将保持为true,然后模糊操作会被取消，如果没有Node需要显示，自然没有必要继续模糊操作。
2. 在此你将操作取消检查包在闭包里，因为一旦Node创建它的Layer或View，那就只能在主线程访问Node的属性。由于你需要访问preventOrCancelDisplay，所以你必须在主线程检查。
3. 最后，确保isCancelledClosure是在主线程进行，无论是在主线程直接运行，还是不再主线程而通过dispatch_sync来调度。它必须是一个同步的调度，因为我们需要闭包完成，并在didCancelblue闭包返回之前设置isCancelled.

在调用applyBlurWithRadius(...)中，修改传递给didCancel的参数，替换一直返回false的闭包为你刚才定义并保留在didCancelBlur的闭包。

```
if let blurredImage = input.applyBlurWithRadius(
  30,
  tintColor: UIColor(white: 0.5, alpha: 0.3),
  saturationDeltaFactor: 1.8,
  maskImage: nil,
  didCancel: didCancelBlur) {
  ...
}
```

编译运行，你看你不会注意到太多差别，但现在任何在cell离开屏幕时还未完成的模糊都会被取消了。这就意味着设备比之前做的更少。你可能观察到轻微的性能提升，特别是在较慢的设备如第三代iPad上运行。

![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0009.png)

你的卡片需要内容，通过下面四个小节，你将学会:

* 创建一个容器Node,它将所有的SubNode绘制到一个单独的CALayer里
* 构建一个Node层次结构
* 创建一个自定义的ASDispalyNode子类，并在后台构建并布局Node层次结构


做完这些，你就会得到一个看起来和添加AsyncDisplayKit之前一样的APP，但有着黄油版顺滑的滑动体验。

##栅格化的容器Node

直到现在，你一直在操作cell内的一个单独的Node，接下来，你将创建一个容器Node，它会包含所有卡片内容。

###添加一个容器Node
继续 RainforestCardCell.swift,在 configureCellDisplayWithCardInfo(cardInfo:) 的backgroundImageNode.imageModificationBlock后面以及Node Layout Section前面添加如下代码：

```
//MARK: Container Node Creation Section
let containerNode = ASDisplayNode()
containerNode.layerBacked = true
containerNode.shouldRasterizeDescendants = true
containerNode.borderColor = UIColor(hue: 0, saturation: 0, brightness: 0.85, alpha: 0.2).CGColor
containerNode.borderWidth = 1
```
这就创建并配置了一个叫做containerNode的ASDisplayNode常量。注意这个容器的shouldRasterizeDescendants,这是一个关于节点如何工作的提示以及一个如何让它们工作的更好的机会.


如单词`descendants (子孙)`所暗示的，你可以创建 AsyncDisplayKit Node的层次结构或树，就如你可以创建Core Animation Layer 的层次结构一样。例如，如果你有一个都是Layer支持的Node层级结构，那么AsyncDisplaykit将会为每个Node创建一个分离的CALayer,Layer层次结构将会和Node层级结构一样，如同镜像。


这听起来很熟悉：它类似于当你使用UIkit时，Layer层次结构镜像于View层次结构。然而，这个Layer的栈有一些不同的效果。

首先，因为是异步渲染，你就不会看到每个layer一个接一个的显示，当AsyncDisplayKit绘制完成每个layer，它马上制作layer的显示内容，所以如果你有一个layer的绘制比其他layer耗时更长，那么它将会在它们之后显示。用户会看到零碎的layer组件，这个过程通常是不可见的，因为Core Animation会在显示任何东西之前重绘所有必须的Layer。

第二，有需要Layer能够引起性能问题。每个CALayer都需要一个支持存储来保存它的像素位图和内容。同样，CoreAnimation必须将每个Layer通过XPC发给渲染服务器。最后，渲染服务器可能需要重绘一些Layer以复合它们，例如在混合layer时，总的来说，更多的Layer意味着CoreAnimation更多的工作。所以限制layer使用的数量有许多不同的好处。

为了解决这个问题，AsyncDisplayKit有一个方便的特性，它允许你绘制一个Node层次结构到一个单独的Layer容器里。这就是shouldRasterizeDescendants所做的，当你设置它，那在完成素有的subnode的绘制之前，ASDisplayNode将不会设置Layer的Contents。

所以在之前的步骤里，设置容器Node的shouldRasterizeDescendants为true有两个好处:

1. 它确保卡片一次显示所有的Node，如同旧的同步绘制
2. 而且它通过栅格化Layer栈为单个Layer并较少未来的合成而提高了效率



不足之处是，由于你将所有的layer放入了一个位图，你就不能再之后单独动画某个Node了。

接下来，在 Container Node Creation Section后，添加backgroundImageNode为containerNode的subnode:

```
//MARK: Node Hierarchy Section
containerNode.addSubnode(backgroundImageNode)
```
>*注意*
>添加Node的顺序很重要，就如同subview和sublayer,最先添加的Node会被之后添加的阻挡显示
>
>


替换 Node Layout Section 的第一行为：


```
//MARK: Node Layout Section
containerNode.frame = FrameCalculator.frameForContainer(featureImageSize: image.size)
```

最后，使用FrameCalculator布局backgroundImageNode:

```
backgroundImageNode.frame = FrameCalculator.frameForBackgroundImage(
  containerBounds: containerNode.bounds)
```
这设置backgroundImageNode填满整个containerNode.

你几乎完成了新的Node层次结构，但首先你需要正确地设置Layer层次结构，因为容器Node现在是根。

###管理容器Node的Layer

在Node Layer and Wrap Up Section，将backgroundImageNode的Layer添加到containerNode的layer上，而不是containerView的Layer上:

```
// Replace the following line...
// self.contentView.layer.addSublayer(backgroundImageNode.layer)
// ...with this line:
self.contentView.layer.addSublayer(containerNode.layer)
```

删除下面的backgroundImageNode保留:

```
self.backgroundImageNode = backgroundImageNode
```

因为cell只需要单独保留容器Node,所以你要移除backgroundImageNode属性。

不再设置cell的contentLayer属性为backgroundImageNode的Layer,现在将其设置为containerNode的layer：

```
// Replace the following line...
// self.contentLayer = backgroundImageNode.layer
// ...with this line:
self.contentLayer = containerNode.layer
```

给RainforestCardCell添加一个可选的ASDisplayNode实例存储为属性containerNode:

```
class RainforestCardCell: UICollectionViewCell {
  var featureImageSizeOptional: CGSize?
  var placeholderLayer: CALayer!
  var backgroundImageNode: ASImageNode?
  var contentLayer: CALayer?
  var containerNode: ASDisplayNode? ///< ADD THIS LINE
  ...
}
```
记住你需要保留你自己的Node，如果你不怎么做它们就会被立即释放。

回到configureCellDisplayWithCardInfo(cardInfo:)，在Node Layer and Wrap Up Section 最后，设置containerNode属性为containerNode常量:

```
self.containerNode = containerNode
```

编译运行，模糊的图形将会再次显示！但还有最后一件事要去改变，因为现在有了新的Node层次结构，回忆之前cell重用时你将图像停止显示。现在你需要让整个Node层次结构停止显示。

###在新的Node层次结构上处理cell重用
继续RainforestCardCell.swift ，在prepareForReuse()里，替换设置backgroundImageNode.preventOrCancelDisplay 为在 containerNode 上调用 recursiveSetPreventOrCancelDisplay(...) 并传递 true：

```
override func prepareForReuse() {
  super.prepareForReuse()
 
  // Replace this line...
  // backgroundImageNode?.preventOrCancelDisplay = true
  // ...with this line:
  containerNode?.recursiveSetPreventOrCancelDisplay(true)
 
  contentLayer?.removeFromSuperlayer()
  ...
}
```

当你要取消整个Node层次结构的绘制，就使用 recursiveSetPreventOrCancelDisplay()。这个方法将会设置这个node以及所有子Node的preventOrCancelDisplay属性，无论true或false。


接下来，依然在prepareForReuse(),用设置containerNode为nil替换设置backgroundImageNode为nil:

```
override func prepareForReuse() {
  ...
  contentLayer = nil

  // Replace this line...
  // backgroundImageNode = nil
  // ...with this line:
  containerNode = nil
}
```

移除RainforestCardCell的backgroundImageNode属性:

```
class RainforestCardCell: UICollectionViewCell {
  var featureImageSizeOptional: CGSize?
  var placeholderLayer: CALayer!
  // var backgroundImageNode: ASImageNode? ///< REMOVE THIS LINE
  var contentLayer: CALayer?
  var containerNode: ASDisplayNode?
  ...
}
```

编译运行，这个APP就如之前一样，但现在你的图像Node在容器Node内，而重用依然和它应有的方式一样.

![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0009.png)

##cell内容
目前为止你有了一个Node层级结构，但容器内还只有一个Node--图像Node.现在是时候设置Node层次结构去复制在添加AsyncDisplayKit之前时应用的视图层次结构了。这意味着添加text和一个未模糊的特征图像。

###添加特征图像

我们要添加特征图像了，它是一个未模糊的图像，显示在卡片的顶部。

打开RainforestCardCell.swift ，并找到configureCellDisplayWithCardInfo(cardInfo:).在Node Creation Section 底部，添加如下代码:

```
let featureImageNode = ASImageNode()
featureImageNode.layerBacked = true
featureImageNode.contentMode = .ScaleAspectFit
featureImageNode.image = image
```
这会创建并配置一个叫做featureImageNode的ASImageNode常量。它被设置为Layer支持的，放大以适用，并设置显示图像，这次不需要模糊。

在Node Hierarchy Section的最后，添加featureImageNode为containerNode的subNode:

```
containerNode.addSubnode(featureImageNode)

```
你正在用更多Node填充容器哦!

在Node Layout Section中，使用FrameCalculator布局featureImageNode：

```
featureImageNode.frame = FrameCalculator.frameForFeatureImage(
  featureImageSize: image.size,
  containerFrameWidth: containerNode.frame.size.width)
```

编译运行，你就会看到特征图像在卡片的顶部出现，位于模糊图像的上方，注意特征图像和模糊图像是如何在同一时间跳出。这是你之前添加的shouldRasterizeDescendants在起作用.

![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0015.png)

##添加Title文本
接下来添加文字Label,以显示动物的名字和描述，首先来添加动物名字吧。

继续configureCellDisplayWithCardInfo(cardInfo:),找到Node Creation Section。添加下列代码到这节尾部，就在创建featureImageNode之后：

```
let titleTextNode = ASTextNode()
titleTextNode.layerBacked = true
titleTextNode.backgroundColor = UIColor.clearColor()
titleTextNode.attributedString = NSAttributedString.attributedStringForTitleText(cardInfo.name)
```

这就创建了一个叫做titleTextNode的AsTextNode常量。

ASTextNode是另一个AsyncDisplayKit提供的Node子类，其用于显示文本。他是一个具有UIlabel效果的Node.它接受一个attributedString,由TextKit支持，有许多特性如文本链接，要学到更多关于这个Node的功能，去看ASTextNode.h吧。

初始羡慕包含一个NSAttributedString的扩展，它提供了一个工厂方法去生成一个属性字符串用于Title和Description文本以显示在雨林卡片上。上面的代码使用了这个扩展的attributedStringForTitleText(...) 方法。

现在在Node Hierarchy Section 底部，添加如下代码:

```
containerNode.addSubnode(titleTextNode)
```
这就添加了titleTextNode到Node层次结构里，它将位于特征图像和背景图像智商，因为它在它们之后添加。


在Node Layout Section底部添加如下代码：

```
titleTextNode.frame = FrameCalculator.frameForTitleText(
  containerBounds: containerNode.bounds,
  featureImageFrame: featureImageNode.frame)
```

一样使用FrameCalculator布局titleTextNode,就像backgroundImageNode和featureImageNode那样。

编译运行，你就有了一个title显示在特征图像的顶部。再次说明，Label只会在整个Cell准备好渲染时才渲染。
![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0017.png)

##添加Description文本
添加一个有着Description文本的Node和添加Title文本的Node类似.

回到configureCellDisplayWithCardInfo(cardInfo:),在 Node Creation Section 最后，添加如下代码。就在之前创建titleTextNode的语句之后:

```
let descriptionTextNode = ASTextNode()
descriptionTextNode.layerBacked = true
descriptionTextNode.backgroundColor = UIColor.clearColor()
descriptionTextNode.attributedString = 
  NSAttributedString.attributedStringForDescriptionText(cardInfo.description)
```

这就创建并配置了一个叫做descriptionTextNode的AStextNode实例。

在 Node Hierarchy Section最后，添加descriptionTextNode到containerNode：

```
containerNode.addSubnode(descriptionTextNode)
```


在 Node Layout Section,一样使用FrameCalculator布局descriptionTextNode：

```
descriptionTextNode.frame = FrameCalculator.frameForDescriptionText(
  containerBounds: containerNode.bounds,
  featureImageFrame: featureImageNode.frame)
```
编译运行，现在你能看到Description文本了。
![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0018.png)

##自定义Node子类
目前为止，你使用了ASImageNode和AStextNode,但有些时候你需要自己定义Node,就如同某些时候在传统的UIKit编程里你需要自己的View一样。

###创建梯度Node类

接下来，你将给GradientView.swift 添加Core Graphics 代码来构建一个自定义的梯度Display Node,这回被用于创建一个绘制梯度的自定义Node.梯度图会显示在特征图像的地步以便让Title看起来更加明显。

打开Layers-Bridging-Header.h，并添加如下代码

```
#import <AsyncDisplayKit/_ASDisplayLayer.h>
```

需要这一步是因为这个类没有包含在主头文件中，你在子类化任何ASDisplayNode或者_ASDisplayLayer时都需要访问这个类。

菜单 File\New\File… 。选择 iOS\Source\Cocoa Touch Class 。命名类为 GradientNode 并使其作为 ASDisplayNode 的子类。选择 Swift 语言并点击 Next 。保存文件再打开 GradientNode.swift 。

添加如下方法到这个类:

```
class func drawRect(bounds: CGRect, withParameters parameters: NSObjectProtocol!,
    isCancelled isCancelledBlock: asdisplaynode_iscancelled_block_t!, isRasterizing: Bool) {

}
```

如同Uiview或者CALayer,你可以子类化ASDisplayNode去做自定义绘制。你可以使用如同用于UIView的Layer或单独的CALayer的绘制代码，这取决于客户Node如何配置Node.查看ASDisplayNode+Subclasses.h 获取更多关于子类化 ASDisplayNode 的信息。

进一步，ASDisplayNode的绘制方法比在UIView和CALayer里接受更多参数，给你提供方法少做工作，更有效率。

要为你的自定义DisplayNode填充内容，你需要实现来自_ASDisplayLayerDelegate协议的drawRect(...) 或 displayWithParameters(...)。在继续之前，看看 _ASDisplayLayer.h 得到这个方法和它们参数的信息。搜索_ASDisplayLayerDelegate。重点看看头文件注释里关于drawRect(..)的描述。


因为梯度图位于特征图的上方，使用Core Graphics 绘制，所以你需要使用drawRect.

打开GradientView.swift 并拷贝drawRect(...)的内容到GradientNode.swift 的drawRect(...)，如下：

```
class func drawRect(bounds: CGRect, withParameters parameters: NSObjectProtocol!,
    isCancelled isCancelledBlock: asdisplaynode_iscancelled_block_t!, isRasterizing: Bool) {
  let myContext = UIGraphicsGetCurrentContext()
  CGContextSaveGState(myContext)
  CGContextClipToRect(myContext, bounds)

  let componentCount: UInt = 2
  let locations: [CGFloat] = [0.0, 1.0]
  let components: [CGFloat] = [0.0, 0.0, 0.0, 1.0,
    0.0, 0.0, 0.0, 0.0]
  let myColorSpace = CGColorSpaceCreateDeviceRGB()
  let myGradient = CGGradientCreateWithColorComponents(myColorSpace, components,
    locations, componentCount)

  let myStartPoint = CGPoint(x: bounds.midX, y: bounds.maxY)
  let myEndPoint = CGPoint(x: bounds.midX, y: bounds.midY)
  CGContextDrawLinearGradient(myContext, myGradient, myStartPoint,
    myEndPoint, UInt32(kCGGradientDrawsAfterEndLocation))

  CGContextRestoreGState(myContext)
}
```

然后删除GradientView.swift,确保编译没有错误

###添加梯度Node
打开RainforestCardCell.swift并找到configureCellDisplayWithCardInfo(cardInfo:)，在Node Creation Section底部，添加如下代码，就在创建descriptionTextNode的代码之后:

```
let gradientNode = GradientNode()
gradientNode.opaque = false
gradientNode.layerBacked = true
```

这就创建了一个叫做gradientNode的GradientNode常量。

在Node Hierarchy Section，在添加featureImageNode那样下面，添加gradientNode到containerNode:

```
//MARK: Node Hierarchy Section
containerNode.addSubnode(backgroundImageNode)
containerNode.addSubnode(featureImageNode)
containerNode.addSubnode(gradientNode) ///< ADD THIS LINE
containerNode.addSubnode(titleTextNode)
containerNode.addSubnode(descriptionTextNode)
```


梯度Node需要这个位置才能在特征图之后，Title之下。

然后添加如下代码到Node Layout Section底部:

```
gradientNode.frame = FrameCalculator.frameForGradient(
  featureImageFrame: featureImageNode.frame)
```

编译运行，你将看到梯度在特征图的底部，title确实看的更清楚了

![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0019.png)


##爆米花特效

如之前提到的，cell的Node内容会在完成绘制时“弹出”，这不是很理想，所以让我们继续，以修复这个问题，但首先，更加深入AsyncDisplayKit以看看它是怎么工作的。

在configureCellDisplayWithCardInfo(cardInfo:) 的Container Node Creation Section，关闭容器Node的shouldRasterizeDescendants：


```
containerNode.shouldRasterizeDescendants = false
```

编译运行，你会注意到现在容器层次结构里不同的Node一个接一个的弹出。你会看到文字弹出然后是特征图，然后是模糊背景图。

当shouldRasterizeDescendants关闭后，AsyncDisplayKit就不是绘制一个容器Layer了，它会创建一个镜像卡片Node层次结构的Layer数。记得爆米花特效存在是因为每个Layer都在它绘制结束后立即出现，而某些Layer比另外一个花费更多时间在绘制上。


这不是我们所需要的，但它描述了AsyncDisplayKit的工作方式，我们不想要这个行为，所以还是将shouldRasterizeDescendants打开：

```
containerNode.shouldRasterizeDescendants = true
```

##在后台构造Node

除了异步的绘制，使用AsyncDisplayKit,你同样可以异步地创建，配置以及布局。深呼吸一下，接下来开始做事情。

###创建一个Node构造操作(OPeration)
你要讲Node层次结构的构造包装到一个NSOperation中，这样做很棒，因为这个操作能很容易的在不同的操作队列中执行，包括后台队列。

打开RainforestCardCell.swift ，然后添加如下方法：

```
func nodeConstructionOperationWithCardInfo(cardInfo: RainforestCardInfo, image: UIImage) -> NSOperation {
  let nodeConstructionOperation = NSBlockOperation()
  nodeConstructionOperation.addExecutionBlock { 
    // TODO: Add node hierarchy construction
  }
  return nodeConstructionOperation
}
```

绘制并不是唯一会拖慢主线程的操作，对于复杂的屏幕，布局计算也有可能变得昂退。目前为止，本教程当前状态的项目，一个缓慢的Node布局会引起Collectionview丢帧。

60FPS意味着你有大约17ms的时间让你的cell准备好显示，否则一个或者多个帧就会被丢掉。这在Table view和 collection view有很复杂的cell时时非常常见的，滑动时丢帧就是这个原因。

AsyncDisplayKit前来救援

你将使用上面的nodeConstructionOperation将所有Node层次结构以及布局从主线程剥离并放入后台NSOperatonQueue，进一步确保Collection view能尽量以接近60 fps的帧率滑动。


>*注意*
>你可以在后台访问并设置Node的属性，但只能在Node的Layer或View被创建之前，也就是当你第一次访问Node的Layer或View属性时。
>


一旦Node的Layer或View被创建，你必须在主线程才能访问和设置Node的属性，因为Node将会转发这些调用到它的Layer或View.如果你得到一个崩溃log说“Incorrect display node thread affinity”,那就意味着在创建Node的Layer或View之后，你依然尝试在后台访问或设置Node的属性。

修改nodeConstructionOperation 操作Block的内容如下:

```
nodeConstructionOperation.addExecutionBlock {
  [weak self, unowned nodeConstructionOperation] in
  if nodeConstructionOperation.cancelled {
    return
  }
  if let strongSelf = self {
    // TODO: Add node hierarchy construction
  }
}
```

在这个操作运行时，cell可能已经被释放了，在那种情况下，你不需要做任何工作。类似的，如果操作被取消了，那一样也没有工作要做了。

之所以对nodeConstructionOperation使用 unowner无主引用是为了避免在操作和执行必要之间产生循环引用。

现在找到configureCellDisplayWithCardInfo(cardInfo:)。将任何在Image Size Section之后的代码移动到nodeConstructionOperation的执行闭包里，将代码放在strongSelf的条件语句里，即 TODO的位置，之后configureCellDisplayWithCardInfo(cardInfo:)将开起来如下:

```
func configureCellDisplayWithCardInfo(cardInfo: RainforestCardInfo) {
  //MARK: Image Size Section
  let image = UIImage(named: cardInfo.imageName)!
  featureImageSizeOptional = image.size
}
```
目前，你会有一些编译错误，这是因为操作Block里的self是weak引用，因此是可选的。但你有一个self的strong引用，因为代码在可选绑定语句内。所以替换错误的几行成下面的样子:

```
strongSelf.contentView.layer.addSublayer(containerNode.layer)
strongSelf.contentLayer = containerNode.layer
strongSelf.containerNode = containerNode
```

最后，添加如下代码到你刚改动的三行之下:

```
containerNode.setNeedsDisplay()
```

编译确保没有错误。如果你现在运行，那么之后占位图会显示，因为Node的创建操作还没有实际使用。让我们来添加它

###使用Node创建操作

打开 RainforestCardCell.swift 并添加如下属性：

```
class RainforestCardCell: UICollectionViewCell {
  var featureImageSizeOptional: CGSize?
  var placeholderLayer: CALayer!
  var backgroundImageNode: ASImageNode?
  var contentLayer: CALayer?
  var containerNode: ASDisplayNode?
  var nodeConstructionOperation: NSOperation? ///< ADD THIS LINE
  ...
}
```

这就添加了一个叫做nodeConstructionOperation可选属性。
当Cell准备回收时，你会使用这个属性取消Node的构造。这会在用户非常快速地滑动Collection View时发生，特别是如果布局还需要一些计算时间的话。

在prepareForReuse()添加如下指示的代码:

```
override func prepareForReuse() {
  super.prepareForReuse()
 
  // ADD FROM HERE...
  if let operation = nodeConstructionOperation {
    operation.cancel()
  }
  // ...TO HERE
 
  containerNode?.recursiveSetPreventOrCancelDisplay(true)
  contentLayer?.removeFromSuperlayer()
  contentLayer = nil
  containerNode = nil
}
```

这就在cell重用时取消了操作，所以如果Node创建还没完成，它也不会完成。

现在找到 configureCellDisplayWithCardInfo(cardInfo:) ,并添加如下指示的代码:

```
func configureCellDisplayWithCardInfo(cardInfo: RainforestCardInfo) {
  // ADD FROM HERE...
  if let oldNodeConstructionOperation = nodeConstructionOperation {
    oldNodeConstructionOperation.cancel()
  }
  // ...TO HERE
 
  //MARK: Image Size Section
  let image = UIImage(named: cardInfo.imageName)!
  featureImageSizeOptional = image.size
}
```

这个Cell现在会在它准备重用并开始配置时，取消任何进行中的 Node构造操作。这确保了操作被取消，即使cell在准备重用前就被重新配置。

在主线程运行

AsyncDisplayKit允许你在非主线程做许多工作。但当它要面对UIKit和CoreAnimation时，你还是需要在主线程做。目前为止，你从主线程移走了所有的Node创建，但还有一件事需要被放在主线程--即设置coreAnimation的Layer层次结构。

在RainforestCardCell.swift里，找到nodeConstructionOperationWithCardInfo(cardInfo:image:) 并替换Node Layer and Wrap Up Section 为如下代码:

```
// 1
dispatch_async(dispatch_get_main_queue()) { [weak nodeConstructionOperation] in
  if let strongNodeConstructionOperation = nodeConstructionOperation {
    // 2
    if strongNodeConstructionOperation.cancelled {
      return
    }
 
    // 3
    if strongSelf.nodeConstructionOperation !== strongNodeConstructionOperation {
      return
    }
 
    // 4
    if containerNode.preventOrCancelDisplay {
      return
    }
 
    // 5
    //MARK: Node Layer and Wrap Up Section
    strongSelf.contentView.layer.addSublayer(containerNode.layer)
    containerNode.setNeedsDisplay()
    strongSelf.contentLayer = containerNode.layer
    strongSelf.containerNode = containerNode
  }
}
```

下面描述一下:

1. 回忆当Node的Layer属性被第一个访问时，所有的Layer会被创建。这就是为何你必须运行Node Layer并在主线程包装小节，因此代码访问Node的Layer.
2. 操作被检查以确定是否在添加Layer之前就已经取消了。在操作完成前，cell被重用或者重新配置，就很可能出现这样的情况，那你就不应该添加Layer了。
3. 作为一个保险，确保Node当前的nodeConstructionOperation和调度闭包的操作是同一个NSOperation
4. 如果containerNode的preventOrCancel是true就立即返回。如果构造操作完成，但node的绘制还没有被取消，你依然不想Node的layer现在在cell里
5. 最后，添加Node的Layer到层次结构中，如果必要，这就创建Layer.

编译确保没有错误

###开始Node创建操作

你依然没有实际创建和开始操作，让我们开始吧

继续在RainforestCardCell.swift里，改变configureCellDisplayWithCardInfo(cardInfo:) 的方法签名为:

```
func configureCellDisplayWithCardInfo(
  cardInfo: RainforestCardInfo,
  nodeConstructionQueue: NSOperationQueue)
```

这里添加了一个新的参数nodeConstructionQueue.它就是一个用于Node创建操作入队的NSOperationQueue.

在func configureCellDisplayWithCardInfo(
  cardInfo: RainforestCardInfo,
  nodeConstructionQueue: NSOperationQueue) 底部，添加如下代码:

```
let newNodeConstructionOperation = nodeConstructionOperationWithCardInfo(cardInfo, image: image)
nodeConstructionOperation = newNodeConstructionOperation
nodeConstructionQueue.addOperation(newNodeConstructionOperation)
```

这就创建了一个Node构造操作，将其保留在nodeConstructionOperation属性，并将其添加到传入的队列。

最后打开 RainforestViewController.swift ，给RainforestViewController添加一个叫做nodeConstructionQueue 的初始化为常量的属性，如下:

```
class RainforestViewController: UICollectionViewController {
  let rainforestCardsInfo = getAllCardInfo()
  let nodeConstructionQueue = NSOperationQueue() ///< ADD THIS LINE
  ...
}
```

接下来，在collectionView(collectionView:cellForItemAtIndexPath indexPath:)里，传递View Controller的 nodeConstructionQueue到 configureCellDisplayWithCardInfo(cardInfo:nodeConstructionQueue:) ：

```
cell.configureCellDisplayWithCardInfo(cardInfo, nodeConstructionQueue: nodeConstructionQueue)
```

cell将会创建一个新的Node构造操作并将其添加到ViewControler的操作队列里并发运行。记住在cell出队时就会创建一个新的Node层次结构。这并不理想，但足够好。如果你要缓存Node的重用，看看ASRangeController 吧

OK，编译运行，你讲看到和之前一样的效果，但现在布局和渲染都没在主线程执行了，牛！我打赌你从来没有想过你看到这一天你所做的事情，这就是AsyncDisplayKit的威力。你可以将更多不需要再主线程操作从主线程中移除，这将给主线程更多机会处理和用户的交互，让你的App摸起来如黄油般顺滑

![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0019.png)



##淡入Cell

在这个就简短的章节，你将会学到:

* 用自定义的Display Layer子类来支持Node
* 触发Node Layer的隐式动画

这将会确保你的移除爆米花特效并最终带来良好的淡入动画

创建一个新的Layer子类.

菜单 File\New\File… ，选择 iOS\Source\Cocoa Touch Class 并单击Next.命名类为AnimatedContentsDisplayLayer并使其作为_ASDisplayLayer子类。选择 Swift语言并单击Next.最后保存并打开AnimatedContentsDisplayLayer.swift .

现在添加如下方法到类:


```
override func actionForKey(event: String!) -> CAAction! {
  if let action = super.actionForKey(event) {
    return action
  }
 
  if event == "contents" && contents == nil {
    let transition = CATransition()
    transition.duration = 0.6
    transition.type = kCATransitionFade
    return transition
  }
 
  return nil
}
```

Layer有一个contents属性，它告诉系统为这个Layer绘制什么，AsyncDisplayKit通过在后台渲染contents并最后在主线程设置contents

这个代码将会添加一个过渡动画，这样contents就会淡入到View中，你可以在Apple的 [Core Animation Prgramming Guide](https://developer.apple.com/library/mac/documentation/cocoa/conceptual/CoreAnimation_guide/ReactingtoLayerChanges/ReactingtoLayerChanges.html)找到更多关于隐式Layer动画


打开 RainforestCardCell.swift。在nodeConstructionOperationWithCardInfo(cardInfo:image:) 里，在Container Node Creation Section开头，改动如下:

```
// REPLACE THIS LINE...
// let containerNode = ASDisplayNode()
// ...WITH THIS LINE:
let containerNode = ASDisplayNode(layerClass: AnimatedContentsDisplayLayer.self)
```

这会告诉容器Node使用AnimatedContentsDisplayLayer实例作为其支持Layer,因此自动带来淡入效果

>*注意*
>只有_ASDisplayLayer的子类才能被异步地绘制
>

编译运行，你讲看到容器Node会在其绘制好之后淡入.


![logo](http://www.raywenderlich.com/wp-content/uploads/2014/10/IMG_0023.png)


##完整工程

[完整工程](https://cdn3.raywenderlich.com/wp-content/uploads/2014/11/Layers-Finished-7.3.zip)请在这里下载!