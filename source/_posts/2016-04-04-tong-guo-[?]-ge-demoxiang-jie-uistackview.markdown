---
layout: post
title: "通过一个Demo详解UIStackView"
date: 2016-04-04 19:48:29 +0800
comments: true
categories: Swift
---

[在上一个章节](http://andrew-anlu.github.io/blog/2016/04/04/uistackviewjie-shao/)我们已经介绍了什么是UIStackView了，其实它更类似于Android开发中的LinerLayer排版技术。

这一章节，我们通过一个完整的例子来讲解UIStackView的用法
<!--more-->
##开始
下下载这个[开始工程](http://www.raywenderlich.com/wp-content/uploads/2015/09/VacationSpots_Starter.zip),下载完毕后，用Iphone6 模拟器运行起来，你将会看到一个度假旅游的列表

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/01-table-view-is-now-correct_750x1334-281x500.png)
点击第一行cell,咋看，这个视图没有什么问题，但是你仔细观察，就会发现有几个问题:

1. 看视图的下面的那一排按钮，它们中间都有一定间隙规则布局，但是它们并没有适配整个屏幕的布局，看着挺丑的，临时转换屏幕landscape orientation,通过 `Command-left`

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/02-issues-visible-in-landscape-view_1334x750-480x270.png)

2. 在详情页面，点击hidden按钮，它成功地隐藏了文字，但是下面的内容并没有顶上去，中间一片空白
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/03-hide-weather-issue_750x1334-281x500.png)

现在你已经有几点好的建议去提升app的体验，现在让我们开始切入这个工程

打开Main.storyboard然后找到 `Spot Info View Controller`,这里有一些颜色在stackView中。

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/04-colorful-scene-in-storyboard_504x636-396x500.png)

这些标签和按钮已经有几种不同颜色的背景色，但是在运行时他们的背景色就是透明的，在这个storyboard中，他们仅仅是为了帮助你展示stackView是怎么改变属性影响嵌套的子视图

你不需要做这些，但是从另一个观点来说你实际上喜欢去看看这些背景色当运行程序的时候，你能临时做些改变在SpotInfoViewController的viewDidLoad()中:

```
// Clear background colors from labels and buttons
for view in backgroundColoredViews {
  view.backgroundColor = UIColor.clearColor()
}
```

当然，其它标签都有占位符文字说明，他们仅仅是为了让你区分哪些是和后台连接的。哪些是描述什么内容的。例如` <whyVisitLabel>`是连接

```
@IBOutlet weak var whyVisitLabel: UILabel!
```
另外一个需要注意的是在这个storyboard中不是默认的 600 x 600,当你使用SizeClass的时候。

SizeClass总是可用的，但是初始化Navigation Controller 默认是总是iPhone 4-inch在模拟器下，这个是容易的在storyboard中，这个模拟器在启动的时候是不受影响的，这个视图将会动态适应不同的设备。
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/05-simulated-metrics-iphone-4-inch_639x173.png)


##你的第一个StackView
第一件事是你将通过一个stackView修复最下面一排按钮的间距，一个stackView能描述在不不同轴向的布局（横向坐标和纵向坐标），其中之一就是子视图之间的距离设置。

幸运的是，修改已经存在的View在一个stackView中并不复杂，选中 `Spot Info View Controller`底部的所有按钮

检查这三个按钮是不是都选择上了，打开左边的控件面板查看，
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/08-verify-button-selection_360x90.png)

一旦选中了，在storyboard的右下角点击new Stack button
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/09-stack_button_outlined_148x52.png)

这个按钮将会变成嵌入式的在stackView中

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/10-bottom-row-is-now-in-stack-view_640x100.png)

这些按钮看起来不是平滑的，稍后我们将会修复

当这个stackView开始嵌套这些按钮的时候，我们将要添加自动布局给这个stackView


当你嵌套一个视图在一个stackView中，这个视图的任何约束都会被移除，例如，在嵌套到stackView之钱，在最前面的那个按钮`Submit Rating`有个垂直距离的约束和` Rating:`label之间:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/11-prior-constraint_420x90.png)


点击` Submit Rating`按钮去看看是否还有这个约束:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/12-no-more-constraints_400x80.png)

为了给stackView添加约束，首先你必须选中它，一个简单的方式去选择这个stackView在outline View:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/140040561762600.png)
另外一种方式是按住 shift&Right-click 在你想选择的视图上，或者按钮 `control`+`shift`+`左键点击`在你想要选择的视图上，你将会看到一个菜单视图，供你选择。
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/15-select-stack-view-in-view-hierarchy-menu_400x280.png)


现在，点击pin按钮在自动布局的工具条上  去 添加约束。

首先检查Constrain to margins，然后添加下载的约束:

	Top: 20, Leading: 0, Trailing: 0, Bottom: 0

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/17-bottom-stack-view-constraints_264x364.png)

现在，这个stackVeiw是正确的尺寸，但是它有一点拉伸第一个按钮，因为它要去利用多余的空间。

stackView有个`distribution`属性来决定子视图的布局，当前，它是fill,这意味着包含的子视图都会完全填充stackview剩下的空间，为了修补这个，这个stackView将要展开其中的一个子视图去填补这个多余的空间

然而，你并不期望这个按钮完全填充stackView，你想让他们占用相同的空间。

选中这个stackView,然后修改它的属性`Distribution`从`Fill`到`Equal Spacing:`
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/19-change-distribution-to-equal-spacing_640x148.png)

现在编译运行，点击这个cell.旋转屏幕，你将会看到底部的按钮平分在屏幕的底部，是不是很酷呢!
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/20-now-buttons-are-equally-spaced_1334x750-480x270.png)

如果不使用stackView来解决这个问题，你不得不使用sapce views，在没两个按钮之间，你的加入等比宽度的约束。很是麻烦。
它看起来像是下面这样，这中间的space View看起来有点灰色


![logo](http://7xkxhx.com1.z0.glb.clouddn.com/21-alternate-solution-1_346x76.png)

这个问题还是不是很大在一个storyboard中，但是很多视图都是动态的，这就不是一个简单的任务了，在运行时去增加一个按钮或者隐藏一个按钮，因为需要去调节视图和约束之间的关系。

为了在一个stack view中隐藏一个视图，你不得不设置子视图的hidden属性为true.现在你将要修复之前说过的那个问题，就是当点击 hidden之后，文字消失，下面的多余空间要顶上去。

##转换Sections

你将要转换所有的section用stack view中，这将要确保你容易的完成你的任务，下一步你将要转换 rating section.


###Rating section
定位到你刚才的页面，然后选择`RATING`标签和 星星的视图：
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/22-select-rating-label-and-stars-label_640x74.png)
然后点击 stack按钮让其嵌套在一个stackView中。
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/23-after-clicking-stack-button_640x74.png)
 现在，点击PIN按钮，添加下面三个属性:
 
 ```
 Top: 20, Leading: 0, Bottom: 20
 ```
 ![logo](http://7xkxhx.com1.z0.glb.clouddn.com/24-add-second-stack-view-constraints_264x171.png)
 
 现在切换到`Attributes inspector`，设置 spacing为8:
 ![logo](http://7xkxhx.com1.z0.glb.clouddn.com/25-set-spacing-to-8_259x87.png)
 
 
 
 这时，你看到视图上的两个控件之间已经有些间距了，
 ![logo](http://7xkxhx.com1.z0.glb.clouddn.com/26-stars-label-weirdly-stretched_640x85.png)
 有时候，xcode可能提示你stackview的位置是不正确的，但是这些警告将会消失，当你更新其它控件的时候，你通常可以忽略他们。
 
为了证明这个，改变 `Alignment `从`Fill`到`Top`然后再改为Fill,你将会看到这个stars 标签变成正确的位置了。

编译运行你的app,一切看起来还是和从前一样.

##取消嵌套一个Stack View
在你深入学习之前，去进行一些"急救"训练，有时，你会发现你的视图上有一个你不再需要的stackview,或许你为了练习而导致的事故。

幸运的是，这里有容易的方式去移除一个嵌套的view从stack view中。

首先，你最好选择你想要删除的stack view,按住`Option`键，然后点击 `stack`按钮，点击 `Unembed `菜单就可以了:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/28-how-to-unembed_186x71.png)


##你的第一个垂直Stack view
现在，你将要创建一个垂直的stack view,选中`WHY VISIT`标签和`<whyVisitLabel>`：
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/29-select-why-visit-labels_640x90.png)
Xcode将会正确的推断出这个视图将会需要一个垂直的stack view,点击`Stack `按钮去嵌套它们到一个stack view中。

当stack view添加成功之后，嵌套的视图的约束将会给删除，当前的这个stack view没有任何约束，所以它会适配子视图中最大尺寸的。

当这个stack view选中的时候，点击 Pin按钮，设置如下属性:
Top, Leading and Trailing 都为0


然后，点击dropdown在右下角，然后选择`WEATHER (current distance = 20):`

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/31-dont-select-nearest-neighbor-constraint_463x417.png)

最后，添加这4个约束，你将会看到如下结果：
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/32-why-visit-stack-view-stretched_640x90.png)
现在你有两个一个展开的stackview,它的右边界是定位到了视图的右边界，然而，这下面的标签依然是同样宽的，你将要修复它通过stack view的`alignment `属性


##Alignment属性

这个alignment属性决定了stack view在其轴向上的布局方式，可能是Fill,Leading,Center和Trailing.

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/33-horizontal-and-vertical-alignment_594x171.png)

在垂直的stackview中，选择不同的属性，将会看到不同的布局:

Fill:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/34-alignment-fill_640x64.png)

Leading:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/35-alignment-leading_640x64.png)

Center:
![lgo](http://7xkxhx.com1.z0.glb.clouddn.com/36-alignment-center_640x64.png)

Trailing:
![lgo](http://7xkxhx.com1.z0.glb.clouddn.com/37-alignment-trailing_640x64.png)

当你测试完每个属性值后，最后设置成Fill

编译运行程序看起来是OK的，特别需要指出的是，`Fill`意味着你想要所有的视图都是完全占用空间在其轴向上，这将引起`WHY VISIT`标签去展开它到右边缘。

但是如果你只想下面的label张开到右边缘，此时该怎么做呢?



##转换"what to see"模块
这个转换和上面的那个很相似，介绍如下:

1. 首先，选择`WHAT TO SEE`标签和`<whatToSeeLabel>`
2. 点击`Stack`按钮
3. 点击`Pin`按钮
4. 设置`margins`约束，添加下面4个约束

```
Top: 20, Leading: 0, Trailing: 0, Bottom: 20
```

5. 设置stack view的Alignment为FIll

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/39-after-what-to-see-section_640x308.png)
编译运行你的工程，验证页面是不是看起来和之前一样。

剩下就是这个`weather `模块了

##转换weather模块
这个weather模块比其他几个稍微复杂一些，因为它包含了一个hidden按钮

一种方法是你将会创建一个最近的stacview通过嵌套`WEATHER`标签和`Hide`按钮在一个水平的stackview中，然后嵌套水平的stackview和`<weatherInfoLabel>`到一个垂直的stackview中。

看起来你像是这样:
![lgo](http://7xkxhx.com1.z0.glb.clouddn.com/40-weather-stack-in-stack_640x92.png)

点击`Stack`按钮:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/43-weather-click-stack-button_640x92.png)

然后点击 Pin按钮，设置margin约束,设置如下:

```
Top: 20, Leading: 0, Trailing: 0, Bottom: 20
```
设置 stack view的Alignment为Fill
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/44-weather-alignment-fill_640x92.png)

你需要一个在hide按钮和`WEATHER `标签的右边加一个约束，因为`WEATHER `标签被加到了stack view中，它的所有约束都被自动去掉了.

然后，你希望底部的`<weatherInfoLabel> `去填充整个stack view.

你可以完成这个通过把`WEATHER `嵌套进一个垂直的stack view中，记住垂直stackview可以设置alignment为 .Leading,假如stack view是拉伸的超出了它固有的边界，它包含的子视图将会到达它的边界。

选择`WEATHER `标签通过document outline，或者通过`Control-Shift-click`方法:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/45-select-just-the-weather-label_640x92.png)

点击 stack按钮:
![lgo](http://7xkxhx.com1.z0.glb.clouddn.com/46-weather-in-horizontal-stack_640x92.png)

设置Alignment为Leading,然后确保axis是垂直方向:
![log](http://7xkxhx.com1.z0.glb.clouddn.com/47-vertical-and-leading_640x92.png)

完美！你已经完成了外部的stackview平铺，在嵌套的stackView中去填充它的宽度，但是内部的stackview允许这个标签去保持它原有的宽度。

编译运行，为什么hide按钮现在飘到上面去了呢？
![dlo](http://7xkxhx.com1.z0.glb.clouddn.com/48-hide-label-incorrect-position_750x573-419x320.png)

它是因为当你嵌套`WEATHER `标签到一个stackview中时，它的所有和hide按钮相关的约束都被移除掉了。

现在你需要给hide按钮和`WEATHER `标签之间增加新的约束,按住`control-drag`从Hide按钮拖向`WEATHER `标签:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/49-drag-to-weather-label_380x94.png)

添加两个约束:

1. Horizontal Spacing
2. Baseline

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/50-add-multiple-constraints_380x224.png)
编译运行，这个Hide按钮看起来正常了。

现在所有的模块都是在唯一的stackview中了，你把他们全部都嵌套进了stackview中。



##设置第一级Stack view
点击 `command` 然后选择所有的5个顶级的stackview在 outline view中
![log](http://7xkxhx.com1.z0.glb.clouddn.com/52-select-all-stack-views-in-outline_640x260.png)

然后点击stack按钮：
![lgo](http://7xkxhx.com1.z0.glb.clouddn.com/53-stack-all-the-views_640x185.png)

点击Pin按钮，设置约束属性:
全部都设置成0.然后设置Spacing为20, Alignment为Fill,你的storyboard看起来像是这样:
![g](http://7xkxhx.com1.z0.glb.clouddn.com/54-set-the-spacing-to-20-and-alignment-to-fill_640x300.png)

编译运行，此时你的 hide按钮又跑偏了，和之前设置的一样，需要把hide和`WEATHER `重新建立约束:
![d](http://7xkxhx.com1.z0.glb.clouddn.com/56-add-constraints-to-button-again_380x223.png)

编译运行，此时hide按钮在正确的位置上了。


##重新布局视图
现在所有的的模块都在顶级的stackview中，你现在可以更改`what to see`模块的位置，比如和`weather `模块的位置进行互换。

选择`middle stack view`从outline view然后拖拽它和weather的那个stackview进行互换，
![log](http://7xkxhx.com1.z0.glb.clouddn.com/57-drag-and-drop-to-reposition-section_639x130.png)

此时，`weather`模块是第三个模块，但是这个 hide按钮不是在stackview中，它不会被移动。

选中 Hide按钮 :
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/58-hide-button-not-moved_640x130.png)
然后点击`Resolve Auto Layout Issues`在自动布局的菜单上点击 update frame:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/59-resolve-auto-layout-issues_356x269.png)



##Size class based configuration
最后，你能把你注意力集中到之前的任务清单上，在加载模式中，垂直空间是昂贵的，所以你想让stackview中的模块靠近些，为了做到这些，你将要使用size classes去设置顶部stackview的空间从20修改成10.

选中顶部的stackview然后点击小小的`+`号，设置spacing:

![log](http://7xkxhx.com1.z0.glb.clouddn.com/61-select-plus-button_260x120.png)

选择 Any Width > Compact Height:

![lgo](http://7xkxhx.com1.z0.glb.clouddn.com/62-anywidth-compact-height_403x108.png)


然后设置Spacing 为10,在 new wAny hC文本框中:
![lgo](http://7xkxhx.com1.z0.glb.clouddn.com/63-set-spacing-to-10_260x160.png)


##动画

打开SpotInfoViewController.swift文件，然后找到`updateWeatherInfoViews(hideWeatherInfo:animated:).`方法

你将要替换这一行代码:

```
weatherInfoLabel.hidden = shouldHideWeatherInfo
```

改成如下:

```
if animated {
  UIView.animateWithDuration(0.3) {
    self.weatherInfoLabel.hidden = shouldHideWeatherInfo
  }
} else {
  weatherInfoLabel.hidden = shouldHideWeatherInfo
}
```

编译运行，点击`hide`和`show`按钮，会不会感觉出来有点动画效果呢?

在stackview中增加动画效果也是很容易的，比如hidden, alignment, distribution, spacing，甚至axis。

##完整工程下载

你可以下载完整的工程 [完整工程](http://www.raywenderlich.com/wp-content/uploads/2015/09/VacationSpots_Complete.zip)


希望能够帮到你~