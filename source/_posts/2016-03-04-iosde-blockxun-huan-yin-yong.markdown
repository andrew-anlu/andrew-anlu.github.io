---
layout: post
title: "ios的Block循环引用"
date: 2016-03-04 13:22:36 +0800
comments: true
categories: iOS
---
ios在开发的过程中，很容易引发内存泄露问题。也很容易造成循环引用，之前使用block的时候也没有过多注意，其实坑很多。
对于新手来说，出现循环引用的时候，很难去排查。
<!--more-->

##Controller之间的block循环引用
现在，我们声明两个类，一个是ViewController,另一个是TLController,在ViewController中有个按钮，点击 push到TlController中。
先看TLController中类的声明：

```
typedef void(^CallbackBlock)();

@interface TLController : UIViewController
- (instancetype)initWithCallback:(CallbackBlock)callback;

@property (nonatomic, copy) CallbackBlock callbackBlock;
```

TlController.m

```
- (instancetype)initWithCallback:(CallbackBlock)callback{
    self=[super init];
    if(self){
        _callbackBlock=callback;
    }
    return self;
}
```
为了验证该类是不是被释放掉了，我们重写两个方法来检测:

```
-(void)viewDidAppear:(BOOL)animated{
    [super viewDidAppear:animated];
    NSLog(@"进入控制器：%@", [[self class] description]);
}
- (void)dealloc {
    NSLog(@"控制器被dealloc: %@", [[self class] description]);
}
```
在 ViewController中，创建一个按钮，按钮的单击事件如下:

```
// 点击button时
- (void)goToNext {
    //__weak __typeof(self) weakSelf=self;
    
    TLController *vc = [[TLController alloc] initWithCallback:^{
        [self.button setTitleColor:[UIColor greenColor] forState:UIControlStateNormal];
    }];
    self.vc = vc;
    [self.navigationController pushViewController:vc animated:YES];
}
```
现在看Viewcontroller，这里就形成了两个循环，因此vc属性得不到释放，如图:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160304-0.png)

这里形成了两个循环

1. ViewContrller->强引用了vc->强引用了callback->强引用了Viewcontroller

2. Viewcontroler->强引用了属性vc->强引用了callback->强引用了Viewcontroller的属性button

我们要解决这两个循环引用，可以如下操作:

不声明vc属性，或者将vc属性声明为weak弱引用类型，在callback回调处，将self.button改为 weakSelf.button,也就是让callback的这个block对viewcontroller弱引用，这样内存就可以顺利释放了。

```
// 点击button时
- (void)goToNext {
    __weak __typeof(self) weakSelf=self;
    
    TLController *vc = [[TLController alloc] initWithCallback:^{
        [weakSelf.button setTitleColor:[UIColor greenColor] forState:UIControlStateNormal];
    }];
   // self.vc = vc;
    [self.navigationController pushViewController:vc animated:YES];
}
```

##Controller和View之间的block引用

我们先定义一个view,用于和Contrller交互，当点击view上的按钮时，就把结果回调给controller;

TLView定义如下:
TlView.h

```
typedef void(^FeedbackBlock)(id model);
@interface TLView : UIView
@property (nonatomic, copy) FeedbackBlock block;
- (instancetype)initWithBlock:(FeedbackBlock)block;
@end
```

TlView.m

```
-(instancetype)initWithBlock:(FeedbackBlock)block{
    self=[super init];
    if(self){
        self.block=block;
        UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
        [button setTitle:@"反馈给controller" forState:UIControlStateNormal];
        button.frame = CGRectMake(50, 200, 200, 45);
        button.backgroundColor = [UIColor redColor];
        [button setTitleColor:[UIColor yellowColor] forState:UIControlStateNormal];
        [button addTarget:self action:@selector(feedback) forControlEvents:UIControlEventTouchUpInside];
        [self addSubview:button];
    }
    return self;
}


- (void)feedback {
    if (self.block) {
        // 传模型回去，这里没有数据，假设传nil
        self.block(nil);
    }
}

/*
// Only override drawRect: if you perform custom drawing.
// An empty implementation adversely affects performance during animation.
- (void)drawRect:(CGRect)rect {
    // Drawing code
}
*/

- (void)dealloc {
    NSLog(@"dealloc: %@", [[self class] description]);
}
```

接下来，在TlController中增加两个属性

	@property (nonatomic, strong) TLView *aView;
    @property (nonatomic, strong) id currentModel;
    
   
  调用如下:
  
  ```
  -(void)testView{
    
   // __weak __typeof(self) weakSelf=self;
    
    self.aView = [[TLView alloc] initWithBlock:^(id model) {
        // 假设要更新model
        self.currentModel = model;
        //weakSelf.currentModel=model;
    }];
    // 假设占满全屏
    self.aView.frame = self.view.bounds;
    [self.view addSubview:self.aView];
    self.aView.backgroundColor = [UIColor whiteColor];
}
  ```
  在viewDidLoad方法中，调用`[self testView]`
  
  这样Controller和view之间就形成了循环引用，如图:
  ![logo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160304-1.png)
  
  1. TlViewController->强引用aView->block->tlViewcontroller属性的currentModel

  解决的办法是:在创建aView的时候，block内对currentModel的引用改成弱引用
  
  ```
  __weak __typeof(self) weakSelf=self;
    
    self.aView = [[TLView alloc] initWithBlock:^(id model) {
        // 假设要更新model
        weakSelf.currentModel=model;
    }];
  ```
  
  很多程序员直接使用_currentModel,其实这样也会造成循环引用，因为_currentModel也是属于类的成员变量，也会被强引用的。要解决此问题，也要改成弱引用
  
  ```
  __block __weak __typeof(_currentModel) weakModel = _currentModel;
self.aView = [[TLView alloc] initWithBlock:^(id model) {
  // 假设要更新model
  weakModel = model;
}];
  ```
  
##模拟循环引用

```
@autoreleasepool {
  A *aVC = [[A alloc] init];
  B *bVC = [[B allcok] init];
  aVC.controller = bVC;
  bVC.controller = aVC;
}
```
  
aVC->强引用了bVC

bVC->强引用了aVC

如果是这样引用，就形成环了。aVC->bVC->aVC，这就形成了环。

###如果一个Controller中，存在一个局部变量，是否循环引用呢?

在Viewcontroller中声明一个变量
`@property (nonatomic,strong)NSMutableArray *array;`



```
-(void)test1{
    self.array = [NSMutableArray arrayWithObjects:@"a",@"b",@"abc",nil];
    TLController *vc = [[TLController alloc] initWithCallback:^{
        [self.array removeObjectAtIndex:0];
    }];
    [self.navigationController pushViewController:vc animated:YES];
}
```
点击跳转按钮，控制台打印

	BlockDemo1[5511:1169090] 进入控制器：TLController
    2016-03-04 14:08:45.612 
点击回退,控制台打印

    BlockDemo1[5511:1169090] 控制器被dealloc: TLController
    
  说明成员变量NSMutableArray不会形成循环引用。



  
  


