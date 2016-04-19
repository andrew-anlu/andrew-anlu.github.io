---
layout: post
title: "我所认识的Quartz2D"
date: 2015-11-26 19:26:28 +0800
comments: true
categories: 
---
##前言
Quartz2D绘图很方便，首先获取到CGcontextRef上下文环境,然后就可以进行绘图了。在自定义UIview的时候，在drawRect()方法中如下调用
 <!-- more -->
 
     CGContextRef ctx=UIGraphicsGetCurrentContext();

drawRect方法在页面初始化的时候会被调用，显示的调用 setNeedsDisplay()方法也会被执行。需要说明的是：手动画图时候它的绘图API的坐标原点位于该控件的左下角，横向为X轴，x坐标越大，位置越向右；纵向为Y轴，Y坐标越大，位置越往下。


###快速入门
用画笔画一个矩形
 
```
- (void)drawRect:(CGRect)rect {
    // Drawing code
    //获取画笔
    CGContextRef ctx=UIGraphicsGetCurrentContext();
    //设置填充颜色
    CGContextSetFillColorWithColor(ctx, [UIColor cyanColor].CGColor);
    //执行 画矩形 命令
    CGContextFillRect(ctx, CGRectMake(100, 100, 200, 200));
}

```

执行如上命令就可以画出来一个矩形，首先或得到画笔，就是CGContextRef，然后设置填充颜色，再执行绘制矩形的命令，就完成了，是不是很简单呢~

绘图的常用API，我整理如下

![draw](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151126-0.png)

###设置绘图属性的常用API

![draw](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151126-1.png)




