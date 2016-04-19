---
layout: post
title: "学习UIBezierPath"
date: 2016-03-03 14:05:08 +0800
comments: true
categories: iOS
---
##前言
之前就看到很多关于`UIBezierPath`的介绍，但是平常开发的时候一直没怎么用过，但是这个`UIBezierPath `对于开发画图是很有用的。故在此整理出来，方便以后使用和查阅。
<!--more-->
使用`UIBezierPath`可以创建基于矢量的路径，此类是Core Graphics框架关于路径的疯转。使用`UIBezierPath`可以定义简单的形状，如三角形，矩形或有多个直线和曲线组成的形状等。


`UIBezierPath `是CGPathRef数据类型的封装。

使用`UIBezierPath `画图的步骤


1. 创建一个`UIBezierPath `对象
2. 调用 -moveToPoint:设置初始线段的起点
3. 添加线或者曲线去定义或者多个子路径
4. 改变`UIBezierPath `对象跟绘图属性。我们可以设置画笔的颜色，size大小等等

##UIBezierPath创建方法

```

+ (instancetype)bezierPath;
+ (instancetype)bezierPathWithRect:(CGRect)rect;
+ (instancetype)bezierPathWithOvalInRect:(CGRect)rect;
+ (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                            cornerRadius:(CGFloat)cornerRadius;
+ (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                        byRoundingCorners:(UIRectCorner)corners 
                              cornerRadii:(CGSize)cornerRadii;
+ (instancetype)bezierPathWithArcCenter:(CGPoint)center 
                                 radius:(CGFloat)radius 
                             startAngle:(CGFloat)startAngle 
                               endAngle:(CGFloat)endAngle 
                              clockwise:(BOOL)clockwise;
+ (instancetype)bezierPathWithCGPath:(CGPathRef)CGPath;
```




      + (instancetype)bezierPath;
      
   这个使用比较多，因为我们可以根据我们的需要任意定制样式，可以画出我们想要的图形
   
    + (instancetype)bezierPathWithRect:(CGRect)rect;
这个工厂方法根据一个矩形画贝塞尔曲线
    
    

    + (instancetype)bezierPathWithOvalInRect:(CGRect)rect;
 这个工厂方法根据一个矩形画内切的贝塞尔曲线，通常用它来画圆和椭圆
 


     + (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                                  cornerRadius:(CGFloat)cornerRadius;
                           
 这个工厂方法画矩形，但是这个矩形是可以画圆角的。第一个参数是矩形，第二个参数是圆角大小
 
 

    + (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                            byRoundingCorners:(UIRectCorner)corners 
                                  cornerRadii:(CGSize)cornerRadii;
 
 这个工厂方法也是画矩形，但是可以指定某一个角成圆角，像这种我们可以很容易的给UIview扩展成圆角了
 
 

    + (instancetype)bezierPathWithArcCenter:(CGPoint)center 
                                     radius:(CGFloat)radius 
                                 startAngle:(CGFloat)startAngle 
                                   endAngle:(CGFloat)endAngle 
                                  clockwise:(BOOL)clockwise;
                                  
 这个工厂方法用于画圆弧，参数如下:
 Center:弧线中心点的坐标
 radius:弧线所在圆的半径
 startAngle:弧线开始的角度值
 endAngle:弧线结束时的角度值
 clockwise:是否顺时针画弧线
 
 
###画三角形
如图:
![三角形](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160303-0.png)

代码:

```
//画三角形
-(void)drawTriangle{
    UIBezierPath *path=[UIBezierPath bezierPath];
    [path moveToPoint:CGPointMake(20, 20)];
    [path addLineToPoint:CGPointMake(self.frame.size.width-40, 20)];
    [path addLineToPoint:CGPointMake(self.frame.size.width/2, self.frame.size.height-20)];
    //最后的闭合线可以通过closePath方法来自动生成。也可以调用addLineToPoint来添加
    
    [path closePath];
    
    //设置线宽
    path.lineWidth=4;
    //设置填充颜色
    UIColor *fillColor=[UIColor yellowColor];
    [fillColor set];
    
    [path fill];
    
    //设置画笔的颜色
    UIColor *stokeColor=[UIColor blueColor];
    [stokeColor set];
    //根据我们设置的各个点，进行连线
    [path stroke];
}
```

###画矩形
![矩形](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160303-1.png)

代码

```
//画矩形
-(void)drawMyRectPath{
    UIBezierPath *path=[UIBezierPath bezierPathWithRect:CGRectMake(100, 100, self.frame.size.width-120, self.frame.size.height-120)];
    path.lineWidth=2;
    path.lineCapStyle=kCGLineCapRound;
    
    //设置填充颜色
    UIColor *fillColor=[UIColor yellowColor];
    [fillColor set];
    [path fill];
    
    //设置画笔颜色
    UIColor *stokeColor=[UIColor blueColor];
    [stokeColor set];
    //根据我们设置的各个点连线
    
    [path stroke];
}
```

###画圆
![ circle](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160303-2.png)

代码:

```
//画圆形或者椭圆
-(void)drawCirlcePath{
    UIBezierPath *path=[UIBezierPath bezierPathWithOvalInRect:CGRectMake(10, 10, self.frame.size.width-20, self.frame.size.height-20)];
    //设置线宽
    path.lineWidth=4;
    //设置填充颜色
    UIColor *fillColor=[UIColor yellowColor];
    [fillColor set];
    
    [path fill];
    
    //设置画笔的颜色
    UIColor *stokeColor=[UIColor blueColor];
    [stokeColor set];
    //根据我们设置的各个点，进行连线
    [path stroke];
    
}
```

###画带圆角的矩形

![圆角矩形](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160303-3.png)

代码

```
//画带圆角的矩形
-(void)drawRoundedRectPath{
    UIBezierPath *path=[UIBezierPath bezierPathWithRoundedRect:CGRectMake(20, 20, self.frame.size.width-40, self.frame.size.height-40) cornerRadius:10];
    
    //设置填充颜色
    UIColor *fillColor=[UIColor yellowColor];
    [fillColor set];
    [path fill];
    
    //设置画笔颜色
    UIColor *stokeColor=[UIColor blueColor];
    [stokeColor set];
    
    //根据我们设置的各个点连线
    [path stroke];
    
}
```

###部分角是圆形的
![circle](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160303-4.png)

只需把上面的代码稍加修改:

```
UIBezierPath *path=nil;
    //[UIBezierPath bezierPathWithRoundedRect:CGRectMake(20, 20, self.frame.size.width-40, self.frame.size.height-40) cornerRadius:10];
    
CGRect rect=CGRectMake(20, 20, self.frame.size.width-40, self.frame.size.height-40);
    
path=[UIBezierPath bezierPathWithRoundedRect:rect byRoundingCorners:UIRectCornerTopLeft cornerRadii:CGSizeMake(20, 20)];
```
 
###画弧度
画弧前，我们需要先了解参考系
![sdf](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160303-5.png)


效果图:

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160303-6.png)

代码:

```
-(void)drawARCPath{
    const CGFloat pi=3.14159;
    CGPoint center=CGPointMake(self.frame.size.width/2, self.frame.size.height/2);
    UIBezierPath *path=[UIBezierPath bezierPathWithArcCenter:center radius:100 startAngle:0 endAngle:kDegreesToRadians(135) clockwise:YES];
    
    path.lineCapStyle=kCGLineCapRound;
    path.lineJoinStyle=kCGLineJoinRound;
    path.lineWidth=5;
    
    UIColor *stokeColor=[UIColor redColor];
    [stokeColor set];
    
    [path stroke];
}
```




  
 