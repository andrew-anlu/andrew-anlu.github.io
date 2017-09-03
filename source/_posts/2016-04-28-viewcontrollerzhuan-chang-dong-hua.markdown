---
layout: post
title: "ViewController转场动画"
date: 2016-04-28 15:54:40 +0800
comments: true
categories: ios
---


##自定义转场动画
ios7中最让我激动的特性之一就是提供了新的API来支持自定义ViewController之间的转场动画。

<!--more-->

在开始研究新的API之间，我们先看看ios7中 navigation controller之间默认的行为发生了那些改变:在navigation controller中，切换两个view controller的动画变得更有交互性。比方说你想要pop一个view controller出去，你可以用手指从屏幕的左边缘开始拖动，慢慢地把当前的viewcontroller向右拖出屏幕去.

接下来，我们来看看这个新API。很有趣，这部分API大量的使用了协议而不是具体的对象。这初看起来有点奇怪，但是我更喜欢这样的设计，因为这种设计给我们这些开发者更大的灵活性。下面，让我们来做件简单的事情:在Navigation Controller中，实现一个自定义的push动画效果，为了完成这个任务，需要实现UINavigationControllerDelegate中的新方法:

```
- (id<UIViewControllerAnimatedTransitioning>)
                   navigationController:(UINavigationController *)navigationController
        animationControllerForOperation:(UINavigationControllerOperation)operation
                     fromViewController:(UIViewController*)fromVC
                       toViewController:(UIViewController*)toVC
{
    if (operation == UINavigationControllerOperationPush) {
        return self.animator;
    }
    return nil;
}
```
从上面的代码可以看出，我们可以根据不同的操作(push或pop)返回不同的animator.我们可以把anmitor存到一个属性中，从而在多个操作之间实现共享，或者我们也可以为每个操作都创建一个新的animator对象，这里的灵活性很多。

为了让动画运行起来，我们创建一个自定义类，并且实现`UIViewControllerAnimatedTransitioning`这个协议:

```
@interface Animator : NSObject <UIViewControllerAnimatedTransitioning>

@end
```

这个协议要求我们实现两个方法，其中一个定义了动画的持续时间:

```
- (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext
{
    return 0.25;
}
```

另一个方法描述整个动画的执行效果 ：

```
- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext
{
    UIViewController* toViewController = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
    UIViewController* fromViewController = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    [[transitionContext containerView] addSubview:toViewController.view];
    toViewController.view.alpha = 0;

    [UIView animateWithDuration:[self transitionDuration:transitionContext] animations:^{
        fromViewController.view.transform = CGAffineTransformMakeScale(0.1, 0.1);
        toViewController.view.alpha = 1;
    } completion:^(BOOL finished) {
        fromViewController.view.transform = CGAffineTransformIdentity;
        [transitionContext completeTransition:![transitionContext transitionWasCancelled]];

    }];

}
```

从上面的例子汇总，你可以看到如何运用洗衣的：这个方法中通过接受一个类型为`id<UIViewControllerContextTransitioning>`的参数，来获取transition context.值的注意的是，执行完动画之后，我们需要调用transitionContext的`completeTransition :`这个方法来更新ViewController的状态。剩下的代码和ios7之前的一样了，我们从transition context 中得到了需要做转场的两个View controller,然后使用最简单的Uiview animation来实现转场动画。这就是全部代码了，我们已经实现了缩放效果的转场动画了。


注意，这里只是为push操作实现了自定义效果的转场动画，对于pop操作，还是会使用默认的滑动效果，另外，上面我们实现的转场动画无法交互，下面我们就来看看解决这个问题。

##交互式的转场动画

想要动画变地交互非常简单，我们只需要覆盖另一个UINavigationControllerDelegate的方法:


```
- (id <UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController*)navigationController
                          interactionControllerForAnimationController:(id <UIViewControllerAnimatedTransitioning>)animationController
{
    return self.interactionController;
}
```

注意，在非交互式动画效果中，该方法返回nil.

这里返回的interaction controller是`UIPercentDrivenInteractionTransition `类的一个实例，开发者不需要任何配置就可以工作。我们创建了一个拖动收拾(Pan REcognizer),下面是处理该手势的代码:

```
if (panGestureRecognizer.state == UIGestureRecognizerStateBegan) {
    if (location.x >  CGRectGetMidX(view.bounds)) {
        navigationControllerDelegate.interactionController = [[UIPercentDrivenInteractiveTransition alloc] init];
        [self performSegueWithIdentifier:PushSegueIdentifier sender:self];
    }
} 
```

只有当用户从屏幕的右半部分开始触摸的时候，我们才把下一次动画效果设置为交互的（通过设置interactionController这个属性来实现），然后执行方法performSegueWithIdentifier:（如果你不是使用的storyboards,那么就直接调用pushViewController...这类方法）。为了让转场动画持续进行，我们需要调用 interaction controller的一个方法:

```
else if (panGestureRecognizer.state == UIGestureRecognizerStateChanged) {
    CGFloat d = (translation.x / CGRectGetWidth(view.bounds)) * -1;
    [interactionController updateInteractiveTransition:d];
} 
```

该方法会根据用户手指拖动的距离计算一个百分比，切换的动画效果也随着这个百分比来走，最酷的是，interaction controller会和animation controller一起协作，我们只使用了简单的UIView animation的动画效果，但是interaction controller却控制了动画的执行进度，我们并不需要吧interaction controller和Animation controller关联起来，因为所有这些系统都以一种解耦的方式自动地替我们完成了。

最后,我们需要更具用户收拾的停止状态来判断该操作是结束还是取消，先调用interaction controller 中对应的方法:


```
else if (panGestureRecognizer.state == UIGestureRecognizerStateEnded) {
    if ([panGestureRecognizer velocityInView:view].x < 0) {
        [interactionController finishInteractiveTransition];
    } else {
        [interactionController cancelInteractiveTransition];
    }
    navigationControllerDelegate.interactionController = nil;
}
```

注意，当切换完成或者取消的时候，记得把interaction controller设置为nil.因为如果下一次的转场是非交互的，我们不应该返回这个旧的interaction controller。

现在我们已经实现了一个完全自定义的可交互的转场动画了。通过简单的手势识别和UIKIT提供的一个类，用几行代码就达到完成了。对于大部分的应用场景，你读到这就够用了，使用上面提到的方法就可以达到你想要的动画效果了。但如果你想更深入了解转场动画或者交互效果进行深度定制，请继续阅读下面的内容。

###完整工程下载
[完整的代码在这里下载](https://github.com/TLiOSDemo/CustomTransitionController/archive/master.zip)

##使用GPUImage定制动画

下面我们就看看如何真正的，彻底的定制动画效果。这一次我们不实用UIviw animation,甚至连Core Animation也不用，完全自己来实现所有的动画效果。

我们使用 [GPUImage](https://github.com/BradLarson/GPUImage)来实现一个非常漂亮的动画效果，这里我们实现的转场动画效果是：两个View controller像素化，然后相互消融在一起。实现方法是先对两个view controller进行截屏，然后再用GPUImage的图片滤镜（filter）处理这两张截图。

首先，我们先创建一个自定义类，这个类实现了UIViewControllerAnimatedTransitioning和UIViewControllerInteractiveTransitioning这两个协议：

```
@interface GPUImageAnimator : NSObject
  <UIViewControllerAnimatedTransitioning,
   UIViewControllerInteractiveTransitioning>

@property (nonatomic) BOOL interactive;
@property (nonatomic) CGFloat progress;

- (void)finishInteractiveTransition;
- (void)cancelInteractiveTransition;

@end
```

为了加速动画的运行，我们可以图片一次加载到GPU中，然后所有的处理和绘图都直接在GPU上执行，不需要再传送到CPU上处理（这种数据传输很慢）。通过使用GPUImageview，我们就可以直接使用OPenGL画图。

创建滤镜链(Filter chain)也非常的直观，我们可以直接在样例代码的setup方法中看到如何构造它。比较有挑战的是如何让滤镜也`动`起来。GPUImage没有直接提供给我们动画效果，因此我们需要每渲染一帧就更新一下滤镜来实现动态的滤镜效果。使用*CADisplayLink*可以完成这个工作：

```
self.displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(frame:)];
[self.displayLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
```
在frame方法中，我们可以根据时间来更新动画进度，并相应地更新滤镜:


```
- (void)frame:(CADisplayLink*)link
{
    self.progress = MAX(0, MIN((link.timestamp - self.startTime) / duration, 1));
    self.blend.mix = self.progress;
    self.sourcePixellateFilter.fractionalWidthOfAPixel = self.progress *0.1;
    self.targetPixellateFilter.fractionalWidthOfAPixel = (1- self.progress)*0.1;
    [self triggerRenderOfNextFrame];
}
```

好了，基本上这样就完成了。如果你想要实现交互式的转场效果，那么在这里，就不能使用时间，而是要根据手势来更新动画进度，其它的代码基本差不多。

这个功能非常强大，你可以使用GPUImage中任何已有的滤镜，或者写一个自己的OpenGL来达到你想要的效果。


##结论
本文只探讨了在navigation controller中的两个view controller之间的转场动画，但是这些做法在tab bar  controller或者任何你自己定义的view controller容器中也是通用的。另外，在ios7中，*UIcollectionViewController*也进行了扩展，现在你可以在布局之间进行自动以及交互的动画切换，背后使用的也是同样的机制。


