---
layout: post
title: "测试并发程序"
date: 2016-04-21 17:38:05 +0800
comments: true
categories: ios
---

在开发高质量应用程序的过程中，测试时一个很重要的工具。在过去，当并发不是应用程序架构中重要组成部分的时候，测试就想单简单。随着这几年的发展，使用并发设计模式变得越来越重要了，想要测试好并发应用程序，已成了一个不小的挑战.

<!--more-->

测试并发代码最主要的困难在于程序或信息流不是反应在调用堆栈上。函数并不会立即返回给调用者，而是通过回调函数block，通知或者一些类似的机制，这些使得测试变得更加困难。

然后，测试异步代码也会带来一些好处，比如可以揭露较差的程序设计，让最终的实现变得更加清晰。


#异步测试的问题
首先，我们来看一个简单地同步单元测试的例子，两个数求和的方法.

```
+ (int)add:(int)a to:(int)b {
    return a + b;
}
```

测试这个方法很简单，只需要比较该方法返回值是否和期望值相同，如果不相同，则测试失败。

```
- (void)testAddition {
    int result = [Calculator add:2 to:2];
    STAssertEquals(result, 4, nil);
}
```

接下来，我们利用block将该方法改成异步返回结果，为了模拟测试失败，我们会在方法实现中故意添加一个bug.

```
+ (int)add:(int)a to:(int)b block:(void(^)(int))block {
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        block(a - b); // 带有bug的实现
    }];
}
```
虽然这是一个人为的例子，但是它却真实的反映了编程中可能经常遇到的问题，只不过实际过程更加复杂罢了。

测试上面的方法最简单的做法就是把断言放到Block中。尽管我们的方法实现中存在bug,但是这种测试永远不是失败的；


```
// 千万不要使用这些代码！
- (void)testAdditionAsync {
    [Calculator add:2 to:2 block:^(int result) {
        STAssertEquals(result, 4, nil); // 永远不会被调用到
    }];
}
```

这里的断言为什么没失败呢？


#关于SenTestingKit

在老版本的xcode中所使用的测试框架是基于 [OCUnit](http://www.sente.ch/software/ocunit/)。为了理解之前所提到的异步测试问题，我们需要了解一下测试包中的各个部分之间的执行顺序。下图展示了一个简化的流程。

![test](http://7xkxhx.com1.z0.glb.clouddn.com/SenTestingKit-call-stack.png)

在测试框架在主run loop开始运行之后，主要执行了一下几个步骤:

1. 配置一个包含所有相关测试的测试包
2. 运行测试包，内部会调用所有以test开头测试用例的方法。运行结束后会返回一个包含单个测试结果的对象。
3. 调用exit()退出测试

这其中我们最感兴趣的是单个测试时如何被调用的。在异步测试中，包含断言的Block会被加到run loop。当所有的测试执行完毕后，测试框架就会退出，而block却从来没有被执行，因此不会引起测试失败。

当然我们有很多种方法来解决这个问题。但是素有的方法都必须在住run loop中运行，而且在测试方法返回和比较结果之前需要处理已入队的所有操作。


[kiwi](https://github.com/allending/Kiwi)使用测试轮询，它可以在测试方法中被滴啊用。[GHUnit](https://github.com/gabriel/gh-unit/)编写了一个单独的测试类，它必须在测试的方法内初始化，并在结束时接受一个通知。以上两种方式都是通过编写相应的代码来确保异步异步测试方法在测试结束之前都不会返回。

#SenTesgingKit的异步扩展

我们对这个问题的解决方案是对SenTestingKit添加一个扩展，它在栈上使用同步执行，并把每个部分加入到主队列中。正如下图所见，在验证整个测试框架结果之前，报告异步测试成功或失败的Block就被加入到队列。这种执行顺序允许我们开启一个测试并等待它的测试结果。

![test](http://7xkxhx.com1.z0.glb.clouddn.com/SenTestingKitAsync-call-stack.png)

如果测试方法以Async结尾，框架就睡认为该方法是异步测试。此外，在异步测试中，我们必须手动的报告测试成功，同时为了防止Block永远不会被调用，我们还需要添加了一个超时方法，之前的错误的测试方法修改如下:


```
- (void)testAdditionAsync {
    [Calculator add:2 to:2 block^(int result) {
        STAssertEquals(result, 4, nil);
        STSuccess(); // 通过调用这个宏来判断是否测试成功
    }];
    STFailAfter(2.0, @"Timeout");
}
```


#设计异步测试

就像同步测试一样，异步测试也应该比被测试的功能简单许多。复杂的测试并不会改进代码的质量，反而会给测试本身带来更多的Bug,在以测试驱动开发的情况下，简单的测试会让我们对组件，接口以及架构的行为有更清醒的认识.

为了运行到实际中，我们创建了一个实例框架: [AsyncTestDemo](sd),它从一个虚拟的服务器获取图像信息，框架中包含了一个资源管理器，它对外提供了一个可以根据图像Id获取图像对象的接口。该接口的工作原理是资源管理器从虚拟服务器获取图片对象信息，并更新到数据库。

虽然这个示例框架只是为了演示，但在我们自己开发的许多应用中也使用了这种模式。

![test](http://7xkxhx.com1.z0.glb.clouddn.com/PinacotecaCore.png)

从上图我们可以知道，示例框架有三个组件我们需要测试:

1. 模型层
2. 模拟服务器请求的服务器接口控制器(API Controller)
3. 管理Core data堆栈以及连接模型层和服务接口控制器的资源管理器


##模型层
测试应该尽量使用同步的方式进行，而模型层就是一个很好的实例。只要不同的被托管对象上下文之间没有复杂的依赖关系，测试用例都应该根据上下文在主线程上设置它自己的core data堆栈，并在其中执行各自的操作。

在这个测试实例中，我们就是在 setup 方法中设置core data堆栈，然后检查 PCImage实体的描述是否存在，如果不存在就构造一个，并更新它的值，当然这和异步测试没有关系，我们就不深入细说了。

##服务器接口控制器
框架中的第二个组件就是服务器接口控制器，它主要处理服务器请求以及服务器API到模型的映射关系。让我们来看一下下面这个方法:

```
- [PCServerAPIController fetchImageWithId:queue:completionHandler:]
```

调用它需要三个形参：一个图片对象Id，所在的执行队列，以及一个完成后的回调方法。

因为服务器根本不存在，一个比较好的做法就是伪造一个代理服务器，正好[OHHTTPStubs](https://github.com/AliSoftware/OHHTTPStubs)可以解决这个问题。在它的最新版本中，可以在示例的请求响应中包含一个bundle,发送给客户端。

为了能stub请求，OHHTTPStubs需要在测试类初始化时或者 setup方法中进行配置。首先我们需要加载一个包含请求响应对象(response)的bundle:

```
NSURL *url = [[NSBundle bundleForClass:[self class]]
                        URLForResource:@"ServerAPIResponses"
                         withExtension:@"bundle"];

NSBundle *bundle = [NSBundle url];
```

然后我们从bundle加载response对象，作为请求的响应值:

```
OHHTTPStubsResponse *response;
response = [OHHTTPStubsResponse responseNamed:@"images/123"
                                   fromBundle:responsesBundle
                                 responseTime:0.1];

[OHHTTPStubs stubRequestsPassingTest:^BOOL(NSURLRequest *request) {
    return YES /* 如果所返回的request是我们所期望的，就返回YES */;
} withStubResponse:^OHHTTPStubsResponse *(NSURLRequest *request) {
    return response;
}];
```

通过如上的设置之后，简化版的[测试服务器接口控制器](https://github.com/objcio/issue-2-async-testing/blob/master/PinacotecaCore/PinacotecaCoreTests/PCServerAPIControllerTests.m)如下:

```
- (void)testFetchImageAsync
{
    [self.server
        fetchImageWithId:@"123"
                   queue:[NSOperationQueue mainQueue]
       completionHandler:^(id imageData, NSError *error) {
          STAssertEqualObjects([NSOperationQueue currentQueue], queue, nil);
          STAssertNil(error, [error localizedDescription]);
          STAssertTrue([imageData isKindOfClass:[NSDictionary class]], nil);

          // 检查返回的字典中的值.

          STSuccess();
       }];
    STFailAfter(2.0, nil);    
}
```

##资源管理器
最后一个部分是资源管理器，它不但把服务器接口控制器和膜形成呢个联系起来，还管理着core data堆栈。下面我们想测试获取一个图片对象的方法:

```
-[PCResourceManager imageWithId:usingManagedObjectContext:queue:updateHandler:]
```

该方法根据Id返回一个图片对象。如果图片在数据库中不存在，它会创建一个包含Id的新对象，然后通过服务器接口控制器获取图片对象的详细信息。

由于资源管理器的测试不应该依赖于服务器接口控制器，所以我们可以用[OCMock](http://ocmock.org/)来模拟，如果要做方法的部分stub,它是一个理想的框架.如以下的 [资源管理器测试](https://github.com/objcio/issue-2-async-testing/blob/master/PinacotecaCore/PinacotecaCoreTests/PCResourceManagerTests.m):

```
OCMockObject *mo;
mo = [OCMockObject partialMockForObject:self.resourceManager.server];

id exp = [[serverMock expect] 
             andCall:@selector(fetchImageWithId:queue:completionHandler:)
            onObject:self];
[exp fetchImageWithId:OCMOCK_ANY queue:OCMOCK_ANY completionHandler:OCMOCK_ANY];
```

上面的代码实际上它并没有真正调用服务器接口控制器的方法，而是调用我们卸载测试类中的方法。

用上面的作坊，对资源管理的测试就变得很直观。当我们调用资源管理器获取资源时，实际上调用的使我们模拟的服务器接口控制器的方法，这样我们也能检查调用服务器接口控制器时参数是否正确。在调用了获取图像对象的方法后，资源管理器会更新模型，然后调用验证测试成功与否的宏。

```
- (void)testGetImageAsync
{
    NSManagedObjectContext *ctx = self.resourceManager.mainManagedObjectContext;
    __block PCImage *img;
    img = [self.resourceManager imageWithId:@"123"
                  usingManagedObjectContext:ctx
                                      queue:[NSOperationQueue mainQueue]
                              updateHandler:^(NSError *error) {
                                       // 检查error是否为空以及image是否已经被更新 
                                       STSuccess();
                                   }];    
    STAssertNotNil(img, nil);
    STFailAfter(2.0, @"Timeout");
}
```


#总结
刚开始的时候，使用并发设计模式测试应用程序是具有一定的挑战性，但是一旦你理解了他们的不同，并建立最佳实践，一切都会变得简单而有趣。

😀fun~


