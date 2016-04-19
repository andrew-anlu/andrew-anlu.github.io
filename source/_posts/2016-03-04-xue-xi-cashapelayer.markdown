---
layout: post
title: "学习CAShapeLayer"
date: 2016-03-04 14:20:24 +0800
comments: true
categories: iOS
---
##前言
CAShapeLayer继承自CALayer,因此，可以使用CaLayer的所有属性。但是，CAShapeLayer需要和贝塞尔曲线配合使用才有意义.

CAShapeLayer是在其坐标系统内绘制贝塞尔曲线的。因此使用CAShapeLayer需要与 `UIbezierPath`一起使用。

它有一个 `path`属性，而`UIBezierPath`就是对 `CGPathRef`类型的封装，因此这两者是绝配
<!--more-->



##CASHapeLayer和drawRect的比较

1. drawRect : 属于CoreGraphics框架，占用cpu,性能消耗大
2. CAShapeLayer:属于CoreAnimation框架，通过GPU来渲染图形，节省性能。动画渲染直接提交给手机的GPU,不消耗内存。

这两者各有各的用途，而不是说有个CAShapeLayer就不需要drawRect了。

##CAShapeLayer月UIBezierPath的关系

1. CAShapeLayer中的shape代表形状的意思，所以需要形状才能生效
2. 贝塞尔曲线可以创建矢量的路径，而`UIBezierPath`类是对 `CGPathRef`的封装
3. 贝塞尔曲线给 `CAShapeLayer`提供路径，CAShapelayer在提供的路径中进行渲染。路径会闭环，所以绘制出了 `Shape`
4. 用于`CAShapeLayer`的贝塞尔曲线作为path,其path是一个首尾相接的闭环的曲线，即使该贝塞尔曲线不是一个闭环的曲线。


##CAShapeLayer与UIBezierPath画圆
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160304-2.png)

代码:

```
-(CAShapeLayer*)drawCircle{
    CAShapeLayer *circleLayer=[CAShapeLayer layer];
    
    circleLayer.frame=CGRectMake(100, 10, 200, 200);
    //设置居中显示
   // circleLayer.position=self.view.center;
    //设置填充颜色
    circleLayer.fillColor=[UIColor clearColor].CGColor;
    //设置线宽
    circleLayer.lineWidth=2;
    //设置线的颜色
    circleLayer.strokeColor=[UIColor redColor].CGColor;
    
    //使用UIBezierPath创建路径
    CGRect frame=CGRectMake(0, 0, 200, 200);
    UIBezierPath *circlePath=[UIBezierPath bezierPathWithOvalInRect:frame];
    //设置CAShapeLayer与UIBezierPath关联
    circleLayer.path=circlePath.CGPath;
    
    [self.view.layer addSublayer:circleLayer];
    
    return circleLayer;
    
}
```

注意，我们这里不是放在-drawRect:方法中调用的。我们直接将这个CAShapeLayer放到self.view.layer上，直接呈现出来。

我们创建一个`CAShapeLayer`，然后配置相关属性，然后再通过 `UIBezierPath`的类方法创建一个内切圆路径，然后将路径指定给`CAShapeLayer.path`,这就将两者关联起来了，最后，将这个层放到了self.view.layer上呈现出来。

##CAShapeLayer与UIBezierPath的简单Loading效果


代码:

```
- (void)drawHalfCircle {
  self.loadingLayer = [self drawCircle];

  // 这个是用于指定画笔的开始与结束点
  self.loadingLayer.strokeStart = 0.0;
  self.loadingLayer.strokeEnd = 0.75;

  self.timer = [NSTimer scheduledTimerWithTimeInterval:0.1
                                                target:self
                                              selector:@selector(updateCircle)
                                              userInfo:nil
                                               repeats:YES];
}

- (void)updateCircle {
  if (self.loadingLayer.strokeEnd > 1 && self.loadingLayer.strokeStart < 1) {
    self.loadingLayer.strokeStart += 0.1;
  } else if (self.loadingLayer.strokeStart == 0) {
    self.loadingLayer.strokeEnd += 0.1;
  }

  if (self.loadingLayer.strokeEnd == 0) {
    self.loadingLayer.strokeStart = 0;
  }

  if (self.loadingLayer.strokeStart >= 1 && self.loadingLayer.strokeEnd >= 1) {
    self.loadingLayer.strokeStart = 0;
    [self.timer invalidate];
    self.timer = nil;
  }
}
``` 
我们要实现这个效果，是通过 `stokeStart`和`stokeEnd`这两个属性好实现的，这两个的取值范围是[0-1],当stokeStart的值慢慢变成1时，我们看到路径是慢慢消失的。这里实现的效果并不好，因为不能一直循环。


