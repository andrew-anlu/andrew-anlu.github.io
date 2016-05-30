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
