---
layout: post
title: "自定义UICollectionViewLayout"
date: 2016-10-28 14:34:50 +0800
comments: true
categories: swift
---


UICollectionView，自从ios6之后就被引入到开发中，现在已经变成了最流程的UI元素之一，它最吸引人的特性就是将数据和布局进行分离，依靠分离的数据元素去处理布局，这个布局对象是决定占位元素和视图的元素。

<!--more-->
你可能已经习惯了默认的flow layout布局，它是一个默认实现的被UIKit,它是由基本的表格布局组成的。当然你也可以自定义布局，你可以按照自己的喜好来对视图进行重排。自定义布局是非常强大和灵活的。

通过本章，你将学会如何自定义的布局，怎么去计算layout Attributes,怎么去处理动态的cell。


## Getting Started

[Download the started project](https://koenig-media.raywenderlich.com/uploads/2015/05/Pinterest-starterproject.zip)，下载后打开

编译运行，效果如图：

![1](https://koenig-media.raywenderlich.com/uploads/2015/05/build_and_run_starter_project.png)


## 创建自定义的布局类
你第一步就是创建一个自定义的布局类，UICollectionView有一个抽象类是`UIcollectionViewLayout`,它定义了你的Collection View 的每个cell的属性集合-`UICollectionViewLayoutAttributes`，它包含了你的CollectionView中每个item的属性，比如frame,透明度等

创建一个类继承自`UICollectionViewLayout `,确保选中的是swift语言，然后创建。

下一步，在你的storyboard中配置下你的自定义布局对象类。

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/storyboard_select_collection_view.png)

然后打开` Attributes Inspector`面板，选中`Custom`，在下一个`class`框中选中你刚才创建的新类`PinterestLayout `：

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/storyboard_change_layout.png)


好了，编译运行：

![3](https://koenig-media.raywenderlich.com/uploads/2015/05/build_and_run_empty_collection.png)

why?

![1](https://koenig-media.raywenderlich.com/uploads/2015/05/meme-nopictures.jpg)

不要慌张！因为你还没有在自定义布局类中写入方法，所以这里什么都不显示。


## 核心布局对象
让我们先看看Collection View的布局调用流程，当Collection View 需要显示一些布局信息的时候，它会询问layout object 去提供一些防范去实现。

![1](https://koenig-media.raywenderlich.com/uploads/2015/05/layout-lifecycle.png)

你自定义的布局必须实现如下方法:

* `prepareLayout()`，在cell将要进入屏幕的时候，这个方法将会被调用，这是一个机会当你去准备展示和计算CollectionView size和坐标的地方
* `collectionViewContentSize()`：在这个方法中，你将会返回CollectionView的高度和宽度-不仅仅是可视的内容，Collection View 将会用这些信息去配置ScrollView的内容尺寸
* `layoutAttributesForElementsInRect(_:): `在这个方法中，你需要返回所有的嵌套在矩形区域中的布局属性，你将要返回一个包含`UICollectionViewLayoutAttributes `属性的集合

ok，现在你知道需要去实现什么方法了-但是如何去计算attributes？


## 计算布局属性






