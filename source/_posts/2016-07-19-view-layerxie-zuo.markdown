---
layout: post
title: "View-Layer协作"
date: 2016-07-19 13:44:10 +0800
comments: true
categories: iOS
---


在ios中，所有的View都是由一个底层的layer来驱动的。View和它的layer之间有着紧密的联系，View其实直接从layer对象中获取了绝大多数它所需要的数据。在ios中也有一些单独的layer,比如`AVCaptureVideoPreviewLayer `和`CAShapeLayer `，它们不需要附加到view上就可以在屏幕上显示内容。两种情况下都是layer起决定作用。当然了，附加到view上的layer和单独的layer在行为上还是稍有不同的。

基本上你改变一个单独的layer的任何属性的时候，都会触发一个从旧值过渡到新值的简单动画（就是所谓的动画`animatable`）。然而，如果你改变的是view中layer的同一个属性，它只会从这一帧直接跳变到下一帧。尽管两种情况中都有layer,但是当layer附加在view上时，它的默认的隐式动画的layer行为就不起作用了。


>*注意*
>animatable 几乎所有的层的属性都是隐性可动画的。你可以在文档中看到它们的简介是以'animatable'结尾的。这不仅包括了比如位置，尺寸，颜色或者透明度这样的绝大多数的数值属性，设置也囊括了像isHidden和doubleSides这样的布尔值。像paths这样的属性也是animatable的。但是它不支持隐式动画。
>
>


在 Core Animation 编程指南的"How to Animate Layer-Backed Views"中，对为什么会这样做出了一个解释:

> UIView默认情况下进制了layer动画，但是在animation block中又重新启用了它们
>
>

这正是我们所看到的额行为，当一个属性在动画block之外被改变时，没有动画，但是当属性在动画block内改变时，就带上了动画。对于这是如何发生的这一问题的答案十分简单和优雅，它优美地阐明和揭示了view和layer之间是如何协同工作和被精心设计的。

无论何时一个可动画的layer属性改变时，layer都会寻找并运行何时的'Action'来实行这个改变。在core Animation的专业术语中就把这样的动画统称为动作(Action,或者CAAction)

>
>CAAction:从技术上来说，这是一个接口，并可以用来做各种事情，但是实际上，某种程度上你可以只把它理解为用来处理动画
>



layer将像文档中缩写的那样去寻找动作，整个过程分为5个步骤。第一步中的view和layer中交互的部分是最有意思的：

layer通过向它的代理发送 `actionForLayer:forKey:`消息来询问提供一个对应属性变化的action.delegate可以通过返回以下三者之一来进行响应:

1. 它可以返回一个动作对象，这种情况下layer将使用这个动作
2. 它可以返回一个nil,这样layer就会到其他地方继续寻找
3. 它可以返回一个NShull对象，告诉layer这里不需要执行一个动作，搜索也会就此停止

而让这一切变得有趣的是，当layer在背后支持一个view的时候，view就是它的delegate;


> 在ios中，如果layer与一个UIview对象关联时，这个属性必须被设置为持有这个layer的那个view
> 


理解这些之后，前一分钟解释起来还复杂无比的现象瞬间就易如反掌了；属性改变时layer会向View请求一个动作，而一般情况下view将返回一个NSNull,只有当属性改变发生在动画block中时，view才会返回实际的动作。哈，但是请别轻信我的这些话，你可以非常容易地验证到底是不是这样。只要对一个一般来说可以动画的layer属性向view询问动作就可以了，比如对于'position':

```
NSLog(@"outside animation block: %@",
      [myView actionForLayer:myView.layer forKey:@"position"]);

[UIView animateWithDuration:0.3 animations:^{
    NSLog(@"inside animation block: %@",
          [myView actionForLayer:myView.layer forKey:@"position"]);
}];
```

运行上面的代码，可以看到在block外view返回的是NSNull对象，而在block中时返回的是一个CABasicAnimation.很优雅，对吧?值得注意的是打印出的 NSNull 是带着一对尖括号的 ("<null>")，这和其他对象一样，而打印 nil 的时候我们得到的是普通括号((null))：

```
outside animation block: <null>
inside animation block: <CABasicAnimation: 0x8c2ff10>
```

对于view中的layer来说，对动作的搜索只会到第一步为止。对于单独的layer来说，剩余的4个步骤可以在 [CALayer](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CALayer_class/Introduction/Introduction.html#//apple_ref/occ/instm/CALayer/actionForKey:)actionForKey: 文档中找到。



## 从UIKit中学习