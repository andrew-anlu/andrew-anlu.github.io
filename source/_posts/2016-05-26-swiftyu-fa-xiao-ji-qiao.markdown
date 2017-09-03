---
layout: post
title: "swift语法小技巧"
date: 2016-05-26 15:09:59 +0800
comments: true
categories: swift
---


###<a name="判断某个对象是否响应了某个方法"> 判断某个对象是否响应了某个方法</a>
<!--more-->

没有参数的:

```
if(view.respondsToSelector(#selector(startAnimating))){
        
}
```

一个参数的:

```
 if(self.modeView.respondsToSelector(#selector(hideAnimated(_:)))){
          
 }
```

多个参数的:

```
 if(self.modeView.respondsToSelector(#selector(setProgress(_:animated:)))){
            
 }
```

第一个参数用`_`，第二个先参数名字，中间没有逗号


### Swift中如何使用 #Debug

在oc中可以直接使用

```
#if DEBUG 
  let a = 2
#else 
  let a = 3
#endif
```
但是swift没有引进预处理指令，但并不代表不可以使用预处理指令。

首先要在 setting中设置，在Swift Compiler - Custom Flags，找到 `Other Swift Flags`,添加如下标记 ` -D DEBUG`,在release模式下添加 `-D RELEASE`,如图:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/661867-43c91f8c973acc77.png)

现在就可以正常的使用#if DEBUG了


### UIColllectionView 

注册UIcollectionViewCell

```
        collectionView.registerClass(NN_XueTangCell.self, forCellWithReuseIdentifier: cellId)

```




