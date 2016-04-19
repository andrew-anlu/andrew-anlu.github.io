---
layout: post
title: "UIStackView介绍"
date: 2016-04-04 15:47:37 +0800
comments: true
categories: swift
---

UIStackView类提供了一个高效的接口用于平铺一行或一列的视图组合.Stack视图使你的依靠自动布局的能力，创建用户接口使得可以动态的调整设备的朝向，屏幕尺寸以及任何可用范围内的变化。Stack视图管理着所有它的 arrageedSubviews属性中视图的布局，这些视图根据它们在arrangedSubviews数组中的顺序沿着stack视图的轴向排列，精确的布局变量根据Stack视图的 axixs,distribution,allignment,spcing,和其它属性决定。

<!--more-->

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/130054395519774.png)

使用 Stack视图，打开一个你希望编辑的storyboard,从对象库中拖出一个Horizontal Stack View或者Vertical Stack View,并放置到你希望的位置上，下一步，将控件或视图拖拽到stack中，如果需要你可以继续添加视图或者控件给指定的statck.Interface buider将根据stack内容自动调节尺寸，你可以通过修改尚需经面板中stack视图的属性调整statck的外观.

##Stack视图与自动布局
Stack视图使用自动布局来定位和控制其管理的视图的尺寸，stack视图沿着它的轴向拼凑第一个和最后一个被管理的视图，使其便捷平齐。对一个月水平stack视图，这意味着第一个被管理的视图的左边界是与stack的左边界对齐的，并且最后一个被管理的视图右边界与stack右边界是平齐的。对于垂直stack,上边界和下边界格式对齐的。如果你设置了stack视图的 `layoutMarginsRelativeArrangement `为YES,stack视图将使用相关的边距与其内容对齐，而不是边界。

对于除去`UIStackViewDistributionFillEqually `分布以外的分布方式，stack视图使用被管理视图的`intrinsicContentSize `属性来计算沿着stack轴向的视图尺寸，`UIStackViewDistributionFillEqually `分布将调节所有被管理视图的在stack轴向上拥有相同尺寸，以填充stack视图。如果可能，stack视图将拉伸所有被管理的视图，来匹配其在stack轴向上最长的原有尺寸。


对于除去`UIStackViewAlignmentFill `对齐以外的对齐方式，stack视图使用其管理的视图的`intrinsicContentSize `属性来计算视图垂直于stack轴向的尺寸。`UIStackViewAlignmentFill `重新调节了所有其管理的视图，使这些视图填充stack视图垂直于其轴向空间。如果可能，stack视图将拉伸所有管理的视图来匹配其垂直于stack轴向的最大固有尺寸。

##定位和调整Stack视图尺寸
当stack视图允许你布局其内容而不直接使用自动布局，你将仍然需要使用自动布局来定位stack视图，通常情况下，这意味着需要皮凑至少两个边界相邻的stack来定义它的位置，没有额外约束的情况下，系统会为stack视图计算一个尺寸来适应其内容:

* 沿着stack视图轴向，其适应尺寸等于其管理的视图尺寸与间距的和
* 垂直于stack视图轴向，其适应尺寸等于其管理的视图中最大视图的尺寸
* 如果stack视图的`layoutMarginsRelativeArrangement `为YES,stack视图的适应尺寸会包括边距空间

你可以提供额外的约束来具体说明stack视图的高度，宽度或者两者兼有，在这些情况下，stack视图调整了其管理的视图的布局和尺寸来填充指定区域。精确的布局变量根据stack视图的属性获得。可以通过查看`UIStackViewDistribution `和`UIStackViewAlignment `枚举，已获得一个完整的stack视图。

你也可以根据stack视图的第一条或最后一条基线定位它，而不是使用顶部，底部或者中心Y值，类似于stack视图的适应尺寸，这些基线都是基于stack视图的内容计算得到的

* 一个水平的stack视图调用 `viewForFirstBaselineLayout `方法或者 `viewForLastBaselineLayout `方法时返回它最高的视图。如果最高的视图也是一个stack视图，那么其返回的将是在嵌套的stack视图上调用`viewForFirstBaselineLayout `方法或者`viewForLastBaselineLayout `方法的结果。

* 一个垂直的stack视图当调用 `viewForFirstBaselineLayout `方法时返回的是其管理的第一个视图，当调用`viewForLastBaselineLayout `方法时返回的是其管理的最后一个视图。如果这两个视图之一也是stack视图，那么其返回的将是在嵌套的stack视图上对应调用`viewForFirstBaselineLayout `方法或`viewForLastBaselineLayout `方法的结果。

>*注意*
>基线对齐方式只作用于那些高度匹配其原本内容高度的视图，如果视图被拉伸或压缩过，那么基线将出现在错误的位置上。
>


##通过Stack视图布局
这有一些通用方法用于stack视图。这个清单是要高亮一些有用的实例来显示 stack视图的灵活性，目前这还不是一个完整的清单。

* 只是定义位置. 你可以通过固定两个与其父视图相邻的边界来定义stack视图的位置。在这里，stack视图的尺寸将根据其管理的视图在两个维度上自由扩展。

举个例子，在Figure 1中，stack视图的左边界和上边界都已经相对固定于其父视图。这些标签将根据带有8个点的两者之间的空间作为第一基准线。这对于相对于其本身左对齐的stack视图内容是有效的。

Figure1定义位置
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/140040561762600.png)


* 定义沿着 stack视图轴向的尺寸。这里，你固定了沿着stack视图轴向相对于其父视图的两个边界，定义了stack视图沿着其轴向的尺寸。你将需要固定其它边界中的一个来定义stack视图的位置。stack视图将沿着其轴向改变尺寸和位置来填充定义的空间:然而，未固定的边界将根据其管理的最大视图的尺寸自动移动。

举例Figure 2,stack视图的左，上，右边界都已经相对于其父视图固定了。使用`UIStackViewDistributionFill `分布是的其内容重设尺寸来填充它的高度，并且从文本框有比标签更低的内容紧凑优先级开始，它将在必要的时候被拉伸。

Figure2定义沿着stack视图轴向的尺寸

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/140059516762099.png)

* 定义垂直于stack视图轴向的尺寸。这类似于上一个实例，但是你固定了垂直于stack视图轴向的连个边界和沿着轴向的一个边界。这使得stack视图在你增加或移除其管理的视图时将沿着其轴向扩展或回缩。除非你使用了
`UIStackViewDistributionFillEqually `分布，被管理的视图将跟根据其原有尺寸调节尺寸。垂直于其轴向的视图将根据其stack视图的对齐模式在其定义的范围内平铺。

举例，Figure3展示了一个包含了四个标签和一个按钮的垂直stack视图。这个stack视图使用了8个点的间隙和`UIStackViewAlignmentCenter `对齐方式。stack视图的高度将根据stack内部元素的增减而增大或回缩。

FIgure3.定义垂直于stack视图轴向的尺寸
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/140110013482874.png)

* 同时定义stack视图的位置和尺寸。这里你固定了stack视图的所有四个边界。stack视图将在提供的范围之内平铺其内容，举例，Figure4展示了一个所有四个边界都相对于其父视图固定的垂直stack视图。通过使用`UIStackViewAlignmentCenter `对齐方式和`UIStackViewDistributionFill `分布方式，stack视图确保其内容将水平和垂直居中填充屏幕，然后，获得想要的布局需要两个额外的步骤，默认情况下，stack视图会垂直拉伸标签而不是图片，要缩放图片控件，就要降低其内容紧凑优先级到低于标签，额外的，为了保持图片缩放时的长宽比，你必须设置图片视图的模式为 Aspect Fit.增加一个图片视图月stack视图间相等约束将有助于确保图片将被缩放来填充可用范围。

Figure 4.同时定义stack视图的位置和尺寸
![logog](http://7xkxhx.com1.z0.glb.clouddn.com/140122324105787.png)

##管理stack视图的展现
UistackView是UIview的非渲染型子类。它没提供其自由的任何用户接口，相反地，它只管理被其管理的视图的位置和尺寸。因此，有些属性(如backgroundCOlor)在stack视图上是无效的。类似的，你无法重写layerClass,drawRect等方法。


这里有一系列的属性来定义stack视图如何平铺其内容。

* axis(轴向)属性决定了stack的朝向，只有垂直和水平
* distributin(分布)属性决定了其管理的视图在沿着其轴向上的布局
* alignment(对齐)属性决定了其管理的视图在垂直于其轴向上的布局
* spacing(空隙)属性决定了其管理的视图间的最小间隙
* baselineRelativeArragement 属性决定了其视图间的垂直间隙是否根据基线测量得到
* layoutMarginsRelativeArrangement 属性决定了 stack 视图平铺其管理的视图时是否要参照它的布局边距

通常情况下，你会使用一个stack视图来布局小数量的视图，你可以通过在其他stack视图上嵌套多个stack视图的方式创建更加复杂的视图层次结构。举例：Figure5展示乐意个包含两个水平stack视图的垂直stack视图。每一个水平stack视图各包含一个标签和一个文本框.

Figure 5.Stack 视图的嵌套
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/140146204108925.png)

你也可以通过增加被管理的视图的额外约束来完备的吊接一个被管理视图的展现，举例说明，你可以使用约束类设置视图的最小或最大的高度或宽度，或者你可以定义一个长宽比。当平铺其内容时，stack视图将使用这些约束。举例来说，在Figure4中，当图片被压缩时，图片视图的一个长宽比约束被强行赋予了一个长宽比数。

>**注意**
>当给一个stack视图内的视图增加约束时要特别注意避免传入冲突，作为惯例，如果一个视图的尺寸在一个指定的维度上默认回到其原本内容的尺寸，那么你可以安全的在这个维度上增加约束
>


##维护其惯例的视图与子视图之间的统一性


Stack视图确保它的arragedSubveiws属性将一直是其 subviews属性的子集合。明确的说，stack视图强制实施了以下规定:

* 无论何时stack视图增加了一个视图到它的arrangedSubviews数组，其也将把这个视图作为子视图增加，如果还未增加的话。
* 无论何时一个子视图从stack视图中移除，那么stack视图也将从从arrangedSubviews数组中移除.
* 从arrangedSubviews移除一个视图并不会将其作为姿势图移除。stack视图将不再管理改视图的尺寸和位置，但是该视图扔将是视图结构的一部分，并且当其可见的狂下仍会被渲染到屏幕上。

当arrangedSubviews数组一直包含着subviews数组的自己和，这些数组间的顺序仍然是独立的。

* arrangedSubviews数组的顺序定义了展现在stack中的视图的顺序。对于水平stack视图，这些视图将以阅读顺序平铺，即最小索引的视图在较大索引视图的左侧。对于垂直stack视图，这些视图是从上到下平铺的，及较小索引的视图在较大索引视图的上方。
* subviews数组中的顺序定义了子视图在z轴上的顺序。如果视图重叠，有较小索引的子视图将出现在有家多音的子视图后方。

##动态改变stack视图内容
当视图被加入，移除或插入arrangedSubviews数组时，或当一个被管理的子视图的hidden属性改变时，stack视图都会自动更新它的布局。

Oc代码如下:

```
// Appears to remove the first arranged view from the stack.
// The view is still inside the stack, it's just no longer visible, and no longer contributes to the layout.
UIView * firstView = self.stackView.arrangedSubviews[0];
firstView.hidden = YES;
```
swift代码如下:

```
// Appears to remove the first arranged view from the stack.
// The view is still inside the stack, it's just no longer visible, and no longer contributes to the layout.
let firstView = stackView.arrangedSubviews[0]
firstView.hidden = true
```
stack视图也会自动响应其任何属性的改变。举例，你可以更新stack视图的axis属性来动态改变朝向

OC代码如下:

```
// Toggle between a vertical and horizontal stack
if (self.stackView.axis == UILayoutConstraintAxisHorizontal) {
    self.stackView.axis = UILayoutConstraintAxisVertical;
}else {
    self.stackView.axis = UILayoutConstraintAxisHorizontal;
}
```

Swift代码如下:

```
// Toggle between a vertical and horizontal stack
if stackView.axis == .Horizontal {
    stackView.axis = .Vertical
}else {
    stackView.axis = .Horizontal
}
```

对于被管理的子视图的hidden属性的变化和stack视图属性的变化，你可以通过将这些改变内置到一个动画块代码的方式以动画的方式展现。

OC代码如下:

```
// Animates removing the first item in the stack.
[UIView animateWithDuration:0.25 animations:^{
    UIView * firstView = self.stackView.arrangedSubviews[0];
    firstView.hidden = YES;
}];
```

Swift代码如下:

```
// Animates removing the first item in the stack.
UIView.animateWithDuration(0.25) { () -> Void in
    let firstView = stackView.arrangedSubviews[0]
    firstView.hidden = true}
```


##常用方法

###创建Stack视图

```
- initWithArrangedSubviews:  (New in iOS 9.0)
```

###管理安排的子视图

```
- addArrangedSubview: (New in iOS 9.0)
  arrangedSubviews Property (New in iOS 9.0)
- insertArrangedSubview:atIndex: (New in iOS 9.0)
- removeArrangedSubview: (New in iOS 9.0)
```

###设置布局

```
alignment Property  (New in iOS 9.0)
axis Property  (New in iOS 9.0)
baselineRelativeArrangement Property  (New in iOS 9.0)
distribution Property  (New in iOS 9.0)
layoutMarginsRelativeArrangement Property  (New in iOS 9.0)
spacing Property  (New in iOS 9.0)
```


###常量

```
UIStackViewDistribution
UIStackViewAlignment
```
