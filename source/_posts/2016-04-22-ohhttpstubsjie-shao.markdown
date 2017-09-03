---
layout: post
title: "OHHTTPStubs介绍"
date: 2016-04-22 17:18:29 +0800
comments: true
categories: iOS
---

[OHHTTPStubs](https://github.com/AliSoftware/OHHTTPStubs)是一个模拟网络请求的一个框架，它使用起来非常方便和强大，它能帮你

1. 测试你的app仿真一个服务器（比如加载一个本地文件）,模拟网络慢的情况等
2. 使用伪造的网络数据编写单元测试

<!--more-->
##简单用法

##在Objc中

```
[OHHTTPStubs stubRequestsPassingTest:^BOOL(NSURLRequest *request) {
  return [request.URL.host isEqualToString:@"mywebservice.com"];
} withStubResponse:^OHHTTPStubsResponse*(NSURLRequest *request) {
  // Stub it with our "wsresponse.json" stub file (which is in same bundle as self)
  NSString* fixture = OHPathForFile(@"wsresponse.json", self.class);
  return [OHHTTPStubsResponse responseWithFileAtPath:fixture
            statusCode:200 headers:@{@"Content-Type":@"application/json"}];
}];
```

##在swift中

```
stub(isHost("mywebservice.com")) { _ in
  // Stub it with our "wsresponse.json" stub file (which is in same bundle as self)
  let stubPath = OHPathForFile("wsresponse.json", self.dynamicType)
  return fixture(stubPath!, headers: ["Content-Type":"application/json"])
}
```

##语法讲解

`OHHTTPStubs stubRequestsPassingTest:`方法创建了模拟服务器，`request `是请求的标准判断，比如

```
[request.URL.host isEqualToString:@"mywebservice.com"]
```
意思就是如果请求的url是`mywebservice.com`，我们就返回response,要不然不执行


```
 NSString* fixture = OHPathForFile(@"wsresponse.json", self.class);
  return [OHHTTPStubsResponse responseWithFileAtPath:fixture
            statusCode:200 headers:@{@"Content-Type":@"application/json"}];
```

返回一个respoinse,其中包含了返回的数据，

1. 获取的一个本地文件
2. 返回的状态码 200  (200表示正确返回)
3. headers：返回的头,如果是json一般是:@{@"Content-Type":@"application/json"}



#快速开始一个Demo


##创建一个空工程
用xcode创建一个工程，用pods创建一个Podfile

内容如下:

```
pod 'OHHTTPStubs', '~> 5.0.0'

target :OHHTTPStubsDemoTests, :exclusive => true do
    link_with 'OHHTTPStubsDemoTests'
    
    pod 'OHHTTPStubs'
end
```

执行 `pod install`

成功安装OHHTTPStubs后，在OHHTTPStubsDemoTests目录中编写测试类:

###创建session
在顶部声明一个变量session

```
@property (nonatomic,strong) NSURLSession *session;
```

在setUp方法中，创建该变量实例:

```
self.session=[NSURLSession sharedSession];
```


在 testExample()方法中：

```
/**
 *  返回自定义的普通文本
 */
- (void)testExample {
    
    //开始模拟服务器
    [OHHTTPStubs stubRequestsPassingTest:^BOOL(NSURLRequest * _Nonnull request) {
        //发送请求的url后缀必须是.com结尾的
        return [request.URL.pathExtension isEqualToString:@"com"];
    } withStubResponse:^OHHTTPStubsResponse * _Nonnull(NSURLRequest * _Nonnull request) {
        //创建一个字符串
        NSData * stubData = [@"hello world" dataUsingEncoding:NSUTF8StringEncoding];
        //响应数据
        /**
         *  responseWithData:返回的数据
            statusCode:状态码,200表示成功
            headers:http的header
         */
        return [OHHTTPStubsResponse responseWithData:stubData statusCode:200 headers:@{@"Content-Type":@"text/plain"}];
    }];
    
    
    //在XCT测试框架中，这个表示期望值，因为这个期望值是支持异步测试的，我们是异步请求，所以一定要是使用XCTestExpectation这个特性
     XCTestExpectation *expectation=[self expectationWithDescription:@"sessionDataTask expectation"];
    
    //创建session任务
    NSURLSessionDataTask *dataTask=[self.session dataTaskWithURL:[NSURL URLWithString:@"hello.com"] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        
        //解析返回的字符串
        NSString *resultStr=[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
        
        NSLog(@"返回的数据:%@",resultStr);
        
        XCTAssert(resultStr!=nil);
        
        //断言返回的字符串是hello world,如果不是，则断言失败
        XCTAssertTrue([resultStr isEqualToString:@"hello world"]);
        
        //在想异步测试的地方加上下面这行代码
        [expectation fulfill];
    }];
    //启动任务
    [dataTask resume];
    
    
    //使用XCTestExpectation,必须设置如下的waitForExpectationsWithTimeout方法，如果超时则失败
    [self waitForExpectationsWithTimeout:4 handler:^(NSError * _Nullable error) {
        if(error){
            NSLog(@"出错了:%s",__FUNCTION__);
        }
    }];
   
    
}
```

这里面使用到了 `XCTestExpectation`，这是特性是用来测试异步程序代码的。在想测试异步代码的地方加上`[expectation fulfill];`,
最后，要加上`waitForExpectationsWithTimeout`方法，它们是配对出现的。



OHHTTPStubs创建了模拟服务器后，下面发的任何网络请求都会被模拟服务器返回，正如上面的代码所示，测试结果:
![test](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160422-3.png)
断言成功！

##添加本地文件

1. 在本地创建一个stub.txt的文本文件，内容自定
2. 在本地创建一个stub.jpg的图片，图片自定

图片
![1](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160422-5.png)

文本
![2](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160422-4.png)


###创建文本的TextStub

```
/**
 *  创建文本的TextStub
 */
-(void)createTextStub{
    // #1
    static id<OHHTTPStubsDescriptor> textStub = nil;
    
    // #2
    textStub= [OHHTTPStubs stubRequestsPassingTest:^BOOL(NSURLRequest * _Nonnull request) {
        return [request.URL.pathExtension isEqualToString:@"txt"];
    } withStubResponse:^OHHTTPStubsResponse * _Nonnull(NSURLRequest * _Nonnull request) {
        
        NSString *path = OHPathForFile(@"stub.txt",self.class);
        
       //#3
        return [[OHHTTPStubsResponse responseWithFileAtPath:path
                                                 statusCode:200 headers:@{@"Content-Type":@"text/plain"}]
                requestTime:1.0f
                responseTime:OHHTTPStubsDownloadSpeedWifi];
        
    }];
    //#4
    textStub.name = @"text stub";
}
```
上面代码意义如下:

1. 声明一个返回的OHHTTPStubsDescriptor,这是创建模拟服务器的返回结果描述
2. 开始创建模拟服务器
3. 返回response，其中设置了返回数据，状态码，请求时间等
4. 给textStub起个名字

##测试文本的模拟服务器
创建一个testStubTextTask方法:

```
/**
 *  测试文本的Stub任务
 */
-(void)testStubTextTask{
    //创建文本的模拟服务器
    [self createTextStub];
    //创建一个期望值
    XCTestExpectation *expection=[self expectationWithDescription:@"high expection"];
    
    NSURLSession *session=[NSURLSession sharedSession];
    
    NSString *urlString=@"stub.txt";
    NSURLSessionDataTask *dataTask = [session dataTaskWithURL:[NSURL URLWithString:urlString] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        
        NSString* receivedText = [[NSString alloc] initWithData:data encoding:NSASCIIStringEncoding];
        NSLog(@"返回的结果:%@",receivedText);
        
        XCTAssert(receivedText!=nil);
        
        [expection fulfill];
    }];
    
    [dataTask resume];
    
    [self waitForExpectationsWithTimeout:5 handler:^(NSError * _Nullable error) {
        if(error){
            NSLog(@"出错了:%@",error.description);
        }
    }];
}
```


##创建图片的stub

```
/**
 *  创建Image的stub
 */
-(void)createImageStub{
    static id<OHHTTPStubsDescriptor> imageStub = nil;
    
    imageStub=[OHHTTPStubs stubRequestsPassingTest:^BOOL(NSURLRequest * _Nonnull request) {
        return [request.URL.pathExtension isEqualToString:@"png"];
    } withStubResponse:^OHHTTPStubsResponse * _Nonnull(NSURLRequest * _Nonnull request) {
        return [OHHTTPStubsResponse responseWithFileAtPath:OHPathForFile(@"stub.jpg", self.class) statusCode:200 headers:@{@"Content-Type":@"image/jpeg"}];
    }];
    
    imageStub.name=@"Image stub";
}
```


##测试Image的模拟服务器

```
/**
 *  测试Image的模拟服务器
 */
- (void)testImageStubTask{
    
    [self createImageStub];
    
    XCTestExpectation *expection=[self expectationWithDescription:@"Image Expection"];
    
    NSURLSessionDataTask *dataStask=[self.session dataTaskWithURL:[NSURL URLWithString:@"test.png" relativeToURL:nil] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        
        UIImage *image=[UIImage imageWithData:data];
        
        NSLog(@"返回的image:%@",image.description);
        
        XCTAssert(image!=nil);
        
        [expection fulfill];
        
    }];
    
    [dataStask resume];
    
    [self waitForExpectationsWithTimeout:3 handler:^(NSError * _Nullable error) {
        if(error){
            NSLog(@"出错了:%@",error.description);
        }
    }];
    
}
```

##完整工程
[完整工程在这里下载](https://github.com/TLOpenSpring/OHHTTPStubsDemo/archive/master.zip)
