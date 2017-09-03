---
layout: post
title: "NSOperationQueue简单介绍"
date: 2016-03-26 08:11:55 +0800
comments: true
categories: iOS
---

在iOS中有两种方式来实现多线程:NSOperation和GCD.
其中GCD是基于C的底层的API,而NSOperation则是GCD实现的Object-c的API,随让NSOPeration是基于GCD实现的，但是并不意味着它是一个GCD的重复版本，相反，我们可以用NSOperation轻易的实现一些GCD要写大量代码的事情，因此，NSOperation是被推荐使用的.

<!--more-->


##为什么优先使用NSOperationQuere,而不是GCD
你可能写过这样的网络请求的代码:

```
dispatch_async(_Queue, ^{  
    //请求数据 
    NSData *data = [NSData dataWithContentURL:[NSURL URLWithString:@"http://domain.com/a.png"]]; 
    dispatch_async(dispatch_get_main_queue(), ^{ 
        [self refreshViews:data];
     });
});
```

这段代码是可以正常工作的，但是有一个很大的缺点，就是*这个任务是无法取消的*


`dataWithContentURL:`是同步拉取数据，它会一直阻塞主线程，直到所有的数据返回之后；这个期间，并发队列就需要为其它任务新建线程，这样可能导致性能下降问题。因此我们不推荐这种写法来从网络上拉取数据。

操作队列是有GCD提供的一个队列模型的Coco对象，GCD提供了更加底层的控制，而操作队列则在GCD之上实现了更加方便的功能。NSOperation相对GCD来说有以下优点:

* 提供了GCD中不那么容易复制的有用特性
* 可以很方便的取消一个NSOperation的执行
* 可以容易的添加任务的依赖关系
* 提供了任务的状态:isExecting,isFinished
*

##NSOperationQueue
NSOperationQueue就是一个线程队列，可以吧 `NSOperation`加入到队列中，可以取消或者执行完队列中所有的 `NSOperation`,我们可以通过`maxConcurrentOperationCount`属性来控制并发任务的数量，当设置为1时，就是一个串行队列。

##NSOperation
它是创建线程的对象，系统已经默认提供了`NSBlockOperaton`和`NSInvocationOperation`.你也可以实现自己的子类，通过重写
`main`或者`start`方法来自定义nsoperation

使用`main`方法非常简单，不需要管理一些状态属性,当main方法返回的时候，这个operation就执行结束了，所以一般用来执行同步任务。

如果你希望拥有更多的控制权，或者想在一个操作中可以执行异步任务，那么重写`start`方法，但是注意，在这种情况下，你必须手动的管理操作的状态，只有发送isFinished的KVO消息时，才认为是operaiton结束。

```

@implementation YourOperation
- (void)start
{
    self.isExecuting = YES;
    // 任务代码 ...
}
- (void)finish //异步回调
{
    self.isExecuting = NO;
    self.isFinished = YES;
}
@end
```

*当实现了`start`方法时，默认就会执行start方法，而不执行main方法*，为了让操作队列捕获到做的改变，需要将状态的属性配合KVO的方式实现，如果你不使用它们默认的sette来进行设置的话，就需要在合适的时候手动发送KVO消息。

需要手动管理的状态有:

1. isExecuting  代表任务正在进行中
2. isFinished   代表任务已经执行完成
3. isCanceled  代表任务已经取消


手动发送KVo消息，通知状态:

```
[self willChangeValueForKey:@"isCancelled"];
_isCancelled = YES;
[self didChangeValueForKey:@"isCancelled"];
```

为了能使用队列所提供的取消功能，你需要在长时间操作中不时地检查isCanceled属性，比如在一个长的循环中:

```
- (void)main
{
    while (notDone && !self.isCancelled) {
        // 任务处理
    }
}
@end
```


##简单使用
NSOperaiton和NSOperationQueue实现多线程的步骤:

1. 先将需要执行的操作封装到一个NSOperation对象中
2. 然后将NSOperation对象添加到NSOperationQueue中
3. 系统会自动将NSOperationQueue中的NSOperation取出来
4. 将取出的NSOperation封装放到一条新线程中执行


NSOperation是个抽象类，并不具备封装操作的能力，必须实现它的子类。

###NSInvocationOperation子类

```
//创建操作对象，封装要执行的任务
//NSInvocationOperation   封装操作
    NSInvocationOperation *operation=[[NSInvocationOperation alloc]initWithTarget:self selector:@selector(test) object:nil];
    
    //执行操作
    [operation start];
```

一旦执行操作，就会调用target的test方法

操作对象默认在主线程中执行，只有添加到列队中才会开启新的线程，即默认情况下，如果操作没有放到队列queue中，都是同步执行，只有将NSoperation放到一个NSOperationQueue中，才会异步执行.

## NSBlockOoperaiton
创建对象和添加操作:

```
//创建NSBlockOperation操作对象
    NSBlockOperation *operation=[NSBlockOperation blockOperationWithBlock:^{
        //......
    }];
    
    //添加操作
    [operation addExecutionBlock:^{
        //....
    }];
```


```
//创建NSBlockOperation操作对象
     NSBlockOperation *operation=[NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"NSBlockOperation------%@",[NSThread currentThread]);
     }];
    
     //添加操作
     [operation addExecutionBlock:^{
        NSLog(@"NSBlockOperation1------%@",[NSThread currentThread]);
     }];
     
     [operation addExecutionBlock:^{
         NSLog(@"NSBlockOperation2------%@",[NSThread currentThread]);
     }];
     
     //开启执行操作
    [operation start];
```

只要NSBlockOperation封装的操作数>1，就会异步操作。


##NSOperationQueue

NSOperationQueue可以调用start方法来执行任务，但默认是同步执行的。
如果将NSOperation添加到NSOperationQueue中，系统就会自动异步执行NSOPeration中的操作。

```

#import "YYViewController.h"

@interface YYViewController ()

@end

@implementation YYViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    //创建NSInvocationOperation对象，封装操作
    NSInvocationOperation *operation1=[[NSInvocationOperation alloc]initWithTarget:self selector:@selector(test1) object:nil];
    NSInvocationOperation *operation2=[[NSInvocationOperation alloc]initWithTarget:self selector:@selector(test2) object:nil];
    //创建对象，封装操作
    NSBlockOperation *operation3=[NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"NSBlockOperation3--1----%@",[NSThread currentThread]);
    }];
    [operation3 addExecutionBlock:^{
        NSLog(@"NSBlockOperation3--2----%@",[NSThread currentThread]);
    }];
    
    //创建NSOperationQueue
    NSOperationQueue * queue=[[NSOperationQueue alloc]init];
    //把操作添加到队列中
    [queue addOperation:operation1];
    [queue addOperation:operation2];
    [queue addOperation:operation3];
}

-(void)test1
{
    NSLog(@"NSInvocationOperation--test1--%@",[NSThread currentThread]);
}

-(void)test2
{
    NSLog(@"NSInvocationOperation--test2--%@",[NSThread currentThread]);
}

@end
```

系统自动将NSOperationqueue中的NSOPeration对象取出，将其封装的操作放到一条新的线程中执行，
上面的代码一共有4个任务，operation1和operation2分别有一个任务，operation3有两个任务，一共4个任务，开启了4条线程。

这些任务是并行执行的。

>**提示**
>队列的取出时有顺序的，与打印结果并不矛盾，这就好比赛跑，起跑的顺序是A,B,C，但是到达终点的顺序就不一样是 A B  C了。
>
>