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

最后