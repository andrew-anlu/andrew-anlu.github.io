---
layout: post
title: "自定义ColletionView布局"
date: 2016-04-24 21:05:32 +0800
comments: true
categories: iOS
---

UICollectionView在ios6中第一次被引入，也是UIKit [视图类中的一颗新星](http://oleb.net/blog/2012/09/uicollectionview/)
。它和UITableview共享一套API设计，但也在UItableView上做了一些扩展。UICOllectionView最强大，同时显著超出UITableView的特色就是其完全灵活的布局结构。这这篇文章中，<!--more--> 我们将会实现一个相当复杂的自定义Collection view布局，并且顺便讨论一下这个类设计的重要部分，项目的实例代码在 [GitHub](https://github.com/objcio/issue-3-collection-view-layouts)上。

##布局对象(Layout Objects)
UITableView和UICollectionView都是 data-source和delegate驱动的。他们在显示其子视图集的过程中仅扮演容器角色,且对子视图集真正的内容毫不知情。

UICollectioNView在此之上进行了进一步抽象。它将子视图的位置，大小和外观的控制权拖过给一个单独的布局对象。通过提供一个自定义布局对象，你技术可以实现任何你能想象到的布局。布局继承自UICollectionVieLayout抽象基类.IOS6中以UICollectionViewFloyLayout类的形式提出了一个具体的布局实现。


我们可以使用flow layout实现一个标准的gridview,这可能是colle tion view中最常见的使用案例了。尽管大多数人都这么想，但是Apple很聪明，没有明确的命名这个类为UIColletionViewGridLayout,而使用了更为通用的术语 flow layout,更好的描述了该类的功能：它通过一个接一个的放置cell来建立自己的布局，当需要的时候，插入横排或竖排的分栏符。通过自定义滚动方向，大小和cell之间的间距，flow layout 也可以在单行或单列中布局cell。实际上，UITableView的布局可以想象成flow layout的一种特殊情况。


在你准备自己写一个UICollectionViewLayout的子类之前，你需要问你自己，你是否能够使用UICollectionViewFlowlayout实现你心里的布局。这个类是[很容易定制的](https://developer.apple.com/library/ios/documentation/uikit/reference/UICollectionViewFlowLayout_class/Reference/Reference.html#//apple_ref/occ/cl/UICollectionViewFlowLayout),并且可以继承本身进行进一步的定制，感兴趣的看[这篇文章](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/UsingtheFlowLayout/UsingtheFlowLayout.html#//apple_ref/doc/uid/TP40012334-CH3-SW4)




##Cells和其他Views

为了适应任意布局,collection view简历了一个类似，但比 table view更灵活的视图层级，像往常一样，你的主要内容显示在cell中，cell可以被任意分组到section中。CollectionView的cell必须是UICollectionViewCell的子类。除了cell，collection view额外管理着两种视图:supplementary views和decoration views.

collection view中的Supplemnetary views相当于table view的section header和footer views.像cells一样，他们的内容都有数据源对象驱动，然而和tableview 中用法不一样，supplementary view并不一定会作为header或footer view;他们的数量和位置完全由布局控制.

Decoration views纯粹为一个装饰品。他们完全属于布局对象，并被布局对象管理，他们并不从dataSource 获取的contents.当布局对象指定需要一个decoration view的时候，collection view会自动创建，并将布局对象提供的布局参数应用到上面去。并不需要为自定义视图准备任何内容。


Supplementary views和decoration views必须是UICollectionReusableView的子类。布局使用的每个视图类都需要在collection view中注册，这样当data Source让它们从reuse pool中出列时，它们才能够创建新的实例。如果你是使用的interface Builder，则可以通过在可视化编辑器中拖拽一个cell到collection view上完成cell在collection view中的注册。同样的方法也可以用在supplementary view上，前提是你使用了UIcollectionviewFlowLayout.如果没有，你只能通过调用 `registerClass:`或者`registerNib:`方法手动注册视图类了。你需要在`viewDidload`中做这些操作.

##自定义布局
作为一个非常有意义的自定义collection view布局的例子，我们不妨想一个典型的日历应用程序中的周视图。日历一次显示一周，星期中的每一天现在列中，每一个日历事件将会在我们的colleectio view中以一个cell显示，位置和大小代表事件起始日期事件和持续时间。

![test](http://7xkxhx.com1.z0.glb.clouddn.com/calendar-collection-view-layout.png)
一般有两种类型的 collection view布局:

1. 独立于内容的布局计算.这正是你所知道的像UITableview和UIcollectionViewFlowLayout这些情况。每个cell的位置和外观不是基于其显示的内容，但所有cell的显示顺序是基于内容的顺序。可以把默认的flow layout作为例子。每个cell都是基于前一个cell的放置(或者如果没有足够的空间，则从下一行开始).布局对象不必访问实际数据来计算布局.
2. 基于内容的布局计算。我们的日历视图正式这样类型的例子。为了计算显示事件的气势和街二叔事件，布局对象需要直接访问 collection view的数据源。在很多情况下，布局对象不仅需要取出当前可见cell的数据，还需要从所有记录中取出一些决定当前那些cell可见的数据。

在我们的日历示例中，布局对象如果访问某一个矩形内的cells的属性，那就必须迭代数据源提供的所有事件来决定那些位于要求的时间窗口个中。与一些相对简单，数据源独立计算的flow layout比起来，这足够计算出cell在一个矩形内的index paths了（假设网格中所有的cells的大小都一样）.

如果有一个依赖内容的布局，那就是暗示你需要些自定义的布局类了，同时不能使用自定义的UICOllectionViewFlowLayout，所以这正是我们需要做的事情。

[UICollectionViewLayout文档](https://developer.apple.com/library/ios/documentation/uikit/reference/UICollectionViewLayout_class/Reference/Reference.html)列出了子类需要重写的方法.


##collectionViewContentSize
由于 collection view对它的content并不知情，所以布局首先要提供的信息就是滚动区域的大小，这样collection view才能正确的管理滚动。布局对象必须在此时计算它内容的总大小，包括supplementary views和decoration views。注意，尽管大多数经典的collection view限制在一个轴方向上滚动(正如UIcollectionviewFlowLayout一样)，但是这不是必须的。

在我们的日历示例中，我们想要视图垂直的滚动。比如，如果我们想要在垂直空间上一个小时占去100点，这样显示一整天的内容高度就是2400点。注意，我们不能够水平滚动，这就意味这我们collectionview只能显示一周。为了能够在日历中的多个星期间分页，我们可以在一个独立的scroll view中（可以使用U[IPageViewController](https://developer.apple.com/library/ios/documentation/uikit/reference/UIPageViewControllerClassReferenceClassRef/UIPageViewControllerClassReference.html)）中使用多个 collection view（一周一个）,或者坚持使用一个collection view并且返回足够大的内容宽度，这回使得用户感觉在两个方向上滑动自由。


```
- (CGSize)collectionViewContentSize
{
    // Don't scroll horizontally
    CGFloat contentWidth = self.collectionView.bounds.size.width;

    // Scroll vertically to display a full day
    CGFloat contentHeight = DayHeaderHeight + (HeightPerHour * HoursPerDay);

    CGSize contentSize = CGSizeMake(contentWidth, contentHeight);
    return contentSize;
}
```

为了简单起见，我选择布局在一个非常简单的模型上：假定每周天数相同，每天时长相同，也就是说天数用0-6表示。在一个真实的日历程序中，布局将会为自己的几段大量使用基于 `NSCalendaar `的日期


##layoutAttributesForElementsInRect：

这是任何布局类中最重要的方法了，同时可能也是最容易让人迷惑的方法。collection view调用这个方法并传递一个自身坐标系统中的矩形过去。这个矩形代表了这个视图的可见矩形区域(也即是它的bounds)，你需要准备好处理传给你的任何矩形。


你的视线必须返回一个包含 UICollectionviewLayoutAttributes对象的数组，为每一个cell包含一个这样的对象，supplementary View或decoration view在矩形区域内是可见的。UICollectionViewLayoutAttributes类包含了colletion view内item的所有相关的布局属性。默认情况下，这个类包含 frame,center,size,transform3D,alpha,Zindex和hidden属性。如果你的布局想要控制其他视图的属性(比如背景颜色)，你可以创建一个UICollectionViewLayoutAttributes的子类，然后加上你自己的属性。

布局属性对象(Layout attributes objects)通过indexPath属性和他们对应的cell,supplementary view或者decoration view关联在一起。collection view为所有items从布局对象中请求到布局属性后，它将会实例化所有视图，并将对应的属性应用到每个视图上去。

注意!这个方法涉及到所有类型的视图，也就是cell,supplementary views和decoration views.一个幼稚的实现可能会选择忽略传入的矩形，并且为collection view中所有的视图返回布局属性。在原型设计和开发布局阶段，这是一个有效的方法。但是，这将会性能产生非常坏的影响，特别是可见cell远少于所有cell数量的时候，collection view和布局对象将会为那些不可见的而试图做额外不必要的工作。

你的视线需要做这几步:

1. 创建一个空的可变数组来存放所有的布局属性
2. 确定index paths中那些cells的frame完全或部分位于矩形中。这个计算需要你从collection view的数据源中取出你需要显示的数据。然后在循环中调用你视线的`layoutattributesForItemIndexPath:`方法为每个index path创建并配置一个合适的布局属性对象，并将每个对象添加到数组中。
3. 如果你的布局包含supplementary views,计算矩形内可见supplementary view的index paths.在循环中调用你实现的`layoutAttributesForSupplementaryViewOfKind:atIndexPath:`,并且将这些对象加到数组中。通过为kind参数传递你选择的不同字符，你可以却分出不同种类的supplementary views(比如headers和footers)。当需要创建视图时，collectionview会将kind字符传回到你的数据源。记住supplermentary 和decoration views的数量和种类完全有布局控制。你不会受到headers和footers的限制.
4. 如果布局包含decoration views，计算矩形内可见decoration views的index paths.在循环中调用你实现的`layoutAttributesForDecorationViewOfKind:atIndexPath: `
，并且将这些对象加到数组中
5. 返回数组

我们自定义的布局没有使用 decoration views,但是使用了两种supplermentary views(column headers和row headers):

```
- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect
{
    NSMutableArray *layoutAttributes = [NSMutableArray array];
    // Cells
    // We call a custom helper method -indexPathsOfItemsInRect: here
    // which computes the index paths of the cells that should be included
    // in rect.
    NSArray *visibleIndexPaths = [self indexPathsOfItemsInRect:rect];
    for (NSIndexPath *indexPath in visibleIndexPaths) {
        UICollectionViewLayoutAttributes *attributes =
        [self layoutAttributesForItemAtIndexPath:indexPath];
        [layoutAttributes addObject:attributes];
    }

    // Supplementary views
    NSArray *dayHeaderViewIndexPaths = [self indexPathsOfDayHeaderViewsInRect:rect];
    for (NSIndexPath *indexPath in dayHeaderViewIndexPaths) {
        UICollectionViewLayoutAttributes *attributes =
        [self layoutAttributesForSupplementaryViewOfKind:@"DayHeaderView"
                               atIndexPath:indexPath];
        [layoutAttributes addObject:attributes];
    }

    NSArray *hourHeaderViewIndexPaths = [self indexPathsOfHourHeaderViewsInRect:rect];
    for (NSIndexPath *indexPath in hourHeaderViewIndexPaths) {
        UICollectionViewLayoutAttributes *attributes =
        [self layoutAttributesForSupplementaryViewOfKind:@"HourHeaderView"
                               atIndexPath:indexPath];
        [layoutAttributes addObject:attributes];
    }
    return layoutAttributes;
}
```




##layoutAttributesFor…IndexPath
有时，collection view会为某个特殊的cell,supplementary 或者decoration view向布局对象请求布局属性，而非所有可见的对象。这就是当其他三个方法开始起作用时，你实现的`layoutAttributesForItemAtIndexPath:`需要创建并返回一个单独的布局属性对象，这样才能正确的格式化传给你的index path 所对应的cell.

你可以通过调用` +[UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:]`这个方法，然后根据index path修改属性。为了得到需要显示在这个index path内的数据，你可能需要访问collection view的数据源。到目前为止，至少确保设置了frame尚需经，除非你所有的cell都位于彼此上方。


```
- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath
{
    CalendarDataSource *dataSource = self.collectionView.dataSource;
    id event = [dataSource eventAtIndexPath:indexPath];
    UICollectionViewLayoutAttributes *attributes =
    [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];
    attributes.frame = [self frameForEvent:event];
    return attributes;
}
```

如果你正在使用自动布局，你可能会赶到惊讶，我们正在直接修改布局参数的frame属性，而不是和约束共事，但这正是UIcollectionViewLayout的工作。尽管你坑你使用自动布局来定义collection view的frame和它内部每个cel的布局，但cells的frames还是需要通过老式的方法计算出来。

类似的，`layoutAttributesForSupplementaryViewOfKind:atIndexPath: `和`layoutAttributesForDecorationViewOfKind:atIndexPath:`方法分别需要为supplementary 和decoration views做相同的事。只有你的布局包含这样的视图你才需要实现这两个方法。UICollectionViewlayoutAttributes包含另外两个工厂方法，

`+layoutAttributesForSupplementaryViewOfKind:withIndexPath:` 和 `+layoutAttributesForDecorationViewOfKind:withIndexPath:`，用他们来创建正确的布局属性对象


##shouldInvalidateLayoutForBoundsChange:
最后，当collection view的bounds改变时，布局需要告诉collection view是否需要重新计算布局。我的猜想是：当collectionview改变大小时，大多数布局会被作废，比如设备旋转的时候。因此，一个优质的实现可能只会简单的返回YES。芮然实现功能很重要，但是scrollview的bounds在滚动时也会改变，这意味着你的布局美妙会被丢弃多次。根据计算的复杂性判断，这将会对性能产生很大的影响。

当collection view的宽度改变时，我们自定义的布局必须被丢弃，但这滚动并不会影响到布局。幸运的是，collection view将它的bounds传给` shouldInvalidateLayoutForBoundsChange: `方法，这样我们便能比较视图当前的bounds和新的bounds来确定返回值

```
- (BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds
{
    CGRect oldBounds = self.collectionView.bounds;
    if (CGRectGetWidth(newBounds) != CGRectGetWidth(oldBounds)) {
        return YES;
    }
        return NO;
}
```

##动画

###插入和删除
UITableview中的cell自带了一套非常漂亮的插入和删除的动画。但是当为UIcollectionView增加和删除cell定义动画功能时，UIKit工程师们遇到这样一个问题：如果Collection view的布局是完全可变的，那么预先定义好的动画就没办法和开发者自定义的布局很好的融合。他们提出了一个优雅的方法：当一个cell(或者supplementary 或者 decoration View)被插入到collection view中时，collection View不仅向其布局请求cell正常正常状态下的布局尚需经，同时还请求其初始的布局尚需经，比如，需要在开始有插入动画的cell。CollectionView会简单的创建一个anmiation block,并在这个block中，将所有cell的属性从初始状态改变到常态

通过提供不同的初始布局属性，你可以完全自定义插入动画。比如设置初始的alpha为0将会产生一个淡入的动画。同时设置一个平移或者缩放将会产生移动缩放的效果。

同样的原理应用到删除上，这次动画是从常态到一些列你设置的最终布局属性。这些逗你需要在布局类中为initial或final布局参数实现的方法:


```
initialLayoutAttributesForAppearingItemAtIndexPath:

initialLayoutAttributesForAppearingSupplementaryElementOfKind:atIndexPath:

initialLayoutAttributesForAppearingDecorationElementOfKind:atIndexPath:

finalLayoutAttributesForDisappearingItemAtIndexPath:

finalLayoutAttributesForDisappearingSupplementaryElementOfKind:atIndexPath:

finalLayoutAttributesForDisappearingDecorationElementOfKind:atIndexPath:
```

##布局间切换
可以通过类似的方式将一个collection view布局动态的切换到另外一个布局。当发送一个`setCollectionViewLayout:animated:`消息时，collection view会为cells在新的布局中查询新的布局参数，然后动态的将每个cell从旧参数变换到新的布局参数。你不需要做任何事情。

##结论
根据自定义collection view布局的复杂性，写一个通常很不容易。确切的说，本质上这和从头写一个完整的实现相同布局自定义视图类一样困难了。因为所涉及的计算需要确定去那些子视图是当前可见的，以及他们的位置。尽管如此，使用UIcollectionview还是给你带来了一些很好的效果，比如cell重用，自动支持动画，更不要提整洁的独立布局，子视图管理。

自定义collection view布局也是向[轻量级view Controller](http://objccn.io/issue-1-1/)迈出了很好的异步，正如你的view controller不要包含任何布局代码。应该和一二个独立的dataSource类结合在一起，collection view的视图控制器将很难再包含任何代码


