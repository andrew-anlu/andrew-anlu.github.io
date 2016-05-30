---
layout: post
title: "ios动画"
date: 2016-05-30 09:26:31 +0800
comments: true
categories: ios
---

我们写的应用程序往往都不是静态的，因为它们需要使用用户的需求以及为执行各种任务而改变状态。

在这些状态之间转换时，清晰的揭示正在发生什么是非常重要的。而不是在页面之间跳跃，动画帮助我们解释用户从哪里来，要到哪里去。

<!--more-->
键盘在View中滑进滑出给了我们一个错觉，让我们以为它是简单的被隐藏在屏幕下方的，并且是手机很自然的一个部分。View Controller转场加强了我们的应用程序的导航结构，并且给了用户正在移向那个方向的提示。微妙的反弹和碰撞使界面栩栩如生，并且激发出了物理的质感。要是没有这些的话，我们就只有一个没有视觉设计的干巴巴的环境了。

动画是叙述你的应用的故事的绝佳方式，在了解动画背景的基本原理之后，设计它们会轻松很多。

## 首要任务
在这篇文章中，我们将特别地针对  Core Anmiation进行探讨，虽然你将看到的很多东西也可以用更高级的UIKit的方法来完成，但是Core Animation将会让你更好的理解正在发生什么。它以一种更明确的方式来描述动画，这对这篇文章以及你自己的代码的读者来说都非常有用。

在看动画如何与我们的屏幕上的看到的内容交互之前，我们需要快速浏览一下Core Animation的`CALayer`，这是动画产生作用的地方。


你大概知道UIView实例，以及layer-backed的NSView,修改它们的layer来委托强大的Core Graphics框架来进行渲染。然而你务必要理解，当把动画添加到一个layer时，是不直接修改它的属性的。

取而代之，Core Animation维护了两个平行的layer层次结构:mode layer tree(模型层树)和presentation layer tree(表示层树)。前者中的layers反映了我们能直接看到的layers的状态，而后者的layers则是正在表现的值的近似。

考虑在view上增加一个渐出动画。如果在动画中的任意时刻，查看layer的opacity值，你是得不到与屏幕内容对应的透明度的。取而代之，你需要查看presentaion layer 以获得正确的结果。

虽然你可能不会去直接设置presentaion layer的属性，但是使用它的当前值来创建新的动画护着在动画发生时与layers交互式非常有用的。

通过使用 `-[CALayer presentaionLayer]`和`[CALayer modelLayer]`，你可以在两个layer之间轻松切换。

## 基本动画
可能最常见的情况是将一个View的属性从一个值改变为另一个值，考虑夏敏的这个例子。

 ![w](http://7xkxhx.com1.z0.glb.clouddn.com/rocket-linear.gif)
 
在这里，我们让红色小火箭的 x-position从77 变为455,刚好超过的parent View的边，为了填充所有路径，我们需要确定我们的火箭在任意时刻所到达的位置。这通常使用线性插值法来完成。

	
	X(T)=x0 + t△x
	
	
也就是说，对于动画给定的一个分数t,火箭的x坐标就是起始点的x坐标77，加上一个到终点的距离∆x = 378,乘以该分数的值。


使用`CABasicAnimation`，我们可以如下实现这个动画:

```
CABasicAnimation *animation = [CABasicAnimation animation];
animation.keyPath = @"position.x";
animation.fromValue = @77;
animation.toValue = @455;
animation.duration = 1;

[rocket.layer addAnimation:animation forKey:@"basic"];

```

请注意我们的动画键路径，也就是position.x,实际上包含一个存储在`position`属性中的CGPoint结构体成员。这是CoreAnimation一个非常方便的特性。请查看[支持的键路径的完整列表](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Key-ValueCodingExtensions/Key-ValueCodingExtensions.html)

然而，当我们运行该代码时，我们意识到火箭在完成动画后马上回到了初始位置，这是因为在默认情况下，动画不会再超出其持续时间后还修改 presentaion layer.实际上，在结束时它甚至会被彻底移除。

一旦动画被移除，presentation layer将回到 model layer的值，并且因为我们从未修改该layer的 postion属性，所以我们的飞船将重新出现在它开始的地方。

这里有两种解决这个问题的方法:

第一种方法是直接在 model layer上更新尚需经，这是推荐的做法，因为它使得动画完全可选。

一旦动画完成并且从layer中移除，presentation layer将回到model layer设置的值，而这个值恰好与动画最后一个步骤相匹配

```
CABasicAnimation *animation = [CABasicAnimation animation];
animation.keyPath = @"position.x";
animation.fromValue = @77;
animation.toValue = @455;
animation.duration = 1;

[rocket.layer addAnimation:animation forKey:@"basic"];

rocket.layer.position = CGPointMake(455, 61);
```

或者，你可以通过设置动画的fillMode属性为`kCAFillModeForward `，并设置`removedOnCompletion `为No以防止它被自动移除:

```
CABasicAnimation *animation = [CABasicAnimation animation];
animation.keyPath = @"position.x";
animation.fromValue = @77;
animation.toValue = @455;
animation.duration = 1;

animation.fillMode = kCAFillModeForward;
animation.removedOnCompletion = NO;

[rectangle.layer addAnimation:animation forKey:@"basic"];
```

如果将已完成的动画保持在layer上时，会造成额外的开销，因为渲染器会去进行额外的绘画工作。

指的指出的是，实际上我们创建的动画对象在被添加到layer时立刻就复制了一份。这个特性在多个view中重用动画时这非常有用。比方说我们想要第二个火箭在第一个火箭起飞后不久后起飞：

```
CABasicAnimation *animation = [CABasicAnimation animation];
animation.keyPath = @"position.x";
animation.byValue = @378;
animation.duration = 1;

[rocket1.layer addAnimation:animation forKey:@"basic"];
rocket1.layer.position = CGPointMake(455, 61);

animation.beginTime = CACurrentMediaTime() + 0.5;

[rocket2.layer addAnimation:animation forKey:@"basic"];
rocket2.layer.position = CGPointMake(455, 111);
```

设置动画的`beginTime`为未来0.5秒将只会影响`rocket2`，因为动画在执行语句`[rocket1.layer addAnimation:animation forKey:@"basic"];`时已经被复制了，并且之后的rocket1也不会考虑对动画对象的改变。

不妨看一看David的[关于动画时间的一篇很棒的文章](http://ronnqvi.st/controlling-animation-timing/)，通过它可以学习如何更精确的控制你的动画。

我决定再使用`CABasicAnimation`的byValule属性创建一个动画，这个动画从presentaion layer的当前值开始，加上byValue的值后结束。这使得动画更易于重用，因为你不需要精确的指定可能无法提前知道的from- 和 toValue的值。

`fromValue`,`byValue`和`toValue`的不同组合可以用来实现不同的效果，如果你需要创建一个可以在你的不同应用中重用的动画，你可以[查看文档](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CABasicAnimation_class/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004496-CH1-SW4)

## 多步动画
这很容易想到一个场景，你想要为你的冻哈定义超过两个步骤，我们可以使用更通用的`CAKeyframeAnimation `，而不是去链接多个`CABasicAnimation `实例。

关键帧(keyFrame)使我们能够定义动画中任意的一个点，然后让core Animation填充所谓的中间帧


比方说我们正在制作我们下一个Iphone应用程序汇总的登录表单，我们希望当用户输入错误的密码时表单会晃动，使用关键帧动画，看起来大概像是这样：
![d](http://7xkxhx.com1.z0.glb.clouddn.com/form.gif)

```
CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
animation.keyPath = @"position.x";
animation.values = @[ @0, @10, @-10, @10, @0 ];
animation.keyTimes = @[ @0, @(1 / 6.0), @(3 / 6.0), @(5 / 6.0), @1 ];
animation.duration = 0.4;

animation.additive = YES;

[form.layer addAnimation:animation forKey:@"shake"];
```

values数组定义了表单应该到哪些位置、

设置keytimes属性让我们能够指定关键帧动画发生的时间。它们被指定为关键帧动画总持续时间的一个分数。

>*注意:*
>
><a style="font-style:normal">
> 请注意我是如何选择不同的值从0到10到-10转换以维持恒定的速度的。
></a>
>
>

设置additive尚需经为YES 使 Core Animation在更新 presentaion layer之前将动画的值添加到 model layer中去。这使得我们能够对所有形式的需要更新的元素重用相同的动画，且无需提前知道它们的位置。因为这个属性从`CAPropertyAnimation `继承，所以你也可以在使用`CABasicAnimation`时使用它。

## 沿路径的动画