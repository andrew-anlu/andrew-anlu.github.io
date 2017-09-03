---
layout: post
title: "深入理解RunLoop"
date: 2016-03-29 17:29:43 +0800
comments: true
categories: iOS
---

![runloop](http://7xkxhx.com1.z0.glb.clouddn.com/1432799466416554.jpeg)

RunLoop是ios和OSX开发中非常基础的一个概念，本章将会介绍一下在ios中，苹果是利用RunLoop实现自动释放池，延迟回调，触摸事件，屏幕刷新等.

##RunLoop的概念
一般来讲，一个线程一次只能执行一个任务，执行完成之后线程就会退出。如果我们需要一个机制，让线程能随时处理事件但并不退出，通常的代码逻辑是这样的:

```
function loop() {
    initialize();
    do {
        var message = get_next_message();
        process_message(message);
    } while (message != quit);
}
```
<!--more-->

 这种模型通常被称为 `Event Loop`,Event Loop在很多系统和框架中都有实现，比如 Node.js的事件处理，比如window程序的消息循环，再比如OS X/IOS里的RunLoop.实现这种模型的关键点在于:如何管理事件/消息,如何让线程在没有处理消息时休眠以避免资源占用，在有消息到来时被唤醒。
 
 所以，RunLoop实际上就是一个对象，这个对象管理了其需要处理的事件和消息，并提供了一个入口函数来执行上面的EventLoop逻辑。线程执行了这个函数后，就会一直处理这个函数内部"接受消息->等待->处理"的循环中，知道这个循环结束(比如传入quit的消息)，函数返回.
 
 在OSX/IOS系统中，提供了两个这样的对象:NSRunLoop和CFRunLoopref.
 
 CFRunLoopRef是在CoreFoundation框架内，它提供了纯C函数的API,所有这些API都是线程安全的。
 
 NSRunLoop是基于CFRunLoopRef的封装，提供了面向对象的API,但是这些API不是线程安全的。
 
## RunLoop与线程的关系

首先，iOS开发中能遇到两个线程对象:pthread_t 和 NSThread.过去苹果有份文档表明了NSThread只是pthread_t 的封装，但那份文档已经失效了，现在它们也有肯定都是直接包装自最底层的mach thread。

你可以通过pthread_main_np() 或 [NSThread mainThread] 来获取主线程,也可以通过pthread_self()或者[NSThread currentThread]来获取当前线程。CFRunLoop是基于pthread来管理的。

苹果不允许直接创建RunLoop,它只提供了两个自动获取的函数:CFRunLoopGetMain()和CFRUnLoopGetCurrent().这两个函数内部的逻辑大概是下面这样:

```
/// 全局的Dictionary，key 是 pthread_t， value 是 CFRunLoopRef
static CFMutableDictionaryRef loopsDic;
/// 访问 loopsDic 时的锁
static CFSpinLock_t loopsLock;
  
/// 获取一个 pthread 对应的 RunLoop。
CFRunLoopRef _CFRunLoopGet(pthread_t thread) {
    OSSpinLockLock(&loopsLock);
     
    if (!loopsDic) {
        // 第一次进入时，初始化全局Dic，并先为主线程创建一个 RunLoop。
        loopsDic = CFDictionaryCreateMutable();
        CFRunLoopRef mainLoop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, pthread_main_thread_np(), mainLoop);
    }
     
    /// 直接从 Dictionary 里获取。
    CFRunLoopRef loop = CFDictionaryGetValue(loopsDic, thread));
     
    if (!loop) {
        /// 取不到时，创建一个
        loop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, thread, loop);
        /// 注册一个回调，当线程销毁时，顺便也销毁其对应的 RunLoop。
        _CFSetTSD(..., thread, loop, __CFFinalizeRunLoop);
    }
     
    OSSpinLockUnLock(&loopsLock);
    return loop;
}
  
CFRunLoopRef CFRunLoopGetMain() {
    return _CFRunLoopGet(pthread_main_thread_np());
}
  
CFRunLoopRef CFRunLoopGetCurrent() {
    return _CFRunLoopGet(pthread_self());
}
```
从上面的代码来看，线程和RunLoop之间是一一对应的，其关系是保存在一个全局的Dictionary里，线程刚创建时并没有RunLoop,如果你不主动获取，那它一直不会有。RunLoop的创建是发生在第一次获取时，RunLoop的销毁时发生在线程结束时，你只能在一个线程的内部获取其RunLoop(主线程除外)


##RunLoop对外的接口

* CFRunLoopRef
* CFRunLoopModelRef
* CFRunLoopSourceRef
* CFRunLoopTimerRef
* CFRunLoopObserverRef

其中，CFRunLoopModelRef类并没有对外暴露，只是通过CFRunLoopRef的接口进行了封装，他们的关系如下:

![runloop](http://7xkxhx.com1.z0.glb.clouddn.com/1432798883604537.png)


一个RunLoop包含若干个Model,每个model又包含若干个Source/Timer/Observer。每次调用RunLoop的主函数时，只能指定其中一个model,这个Model被称作为CurrentMode.如果需要切换Mode,只能退出Loop,再重新指定一个Mode进入。这样做是为了分隔开不同组的 Source/Timer/Observer,让其互不影响.

CFRunLoopSourceREf是事件产生的地方。Source有两个版本，Source0和Source1.

* Source0只包含了一个回调（函数指针），它并不能主动触发事件。使用时，你需要先调用CFRunLoopSourceSignal(source)，将这个Source标记为待处理，然后手动调用CFRunLoopwakeUp(runloop)来唤醒RunLoop,让其处理这个事件
* Source1包含乐业一个match_port和一个回调(函数指针),被用于通过内核和其它线程相反发送消息。这种Source能主动唤醒Runloop的线程

*CFRUnLoopTimerRef*是基于时间的触发器，它和NStimer可以混用，其包含一个时间长度和一个回调(函数指针).当其加入到RUnLoop时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调.

*CFRunLoopObserverRef*是观察者，每个Observer都包含了一个回调，当Runloop的状态发生变化时，观察者就能通过回调接收到这个变化。可以观测的时间点有以下几个:


```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```

上面的 Source/Timers/Observer 被统称为Mode item,一个Item可以被同事加入多个Mode,但一个Item被重复加入同一个mode时是不会有效果的。如果一个Mode中一个item都没有，则RunLoop会直接退出，不进入循环。
##Runloop的Mode

CFRunLoopMode和CFRunLoop的结构大致如下:

```
struct __CFRunLoopMode {
    CFStringRef _name;            // Mode Name, 例如 @"kCFRunLoopDefaultMode"
    CFMutableSetRef _sources0;    // Set
    CFMutableSetRef _sources1;    // Set
    CFMutableArrayRef _observers; // Array
    CFMutableArrayRef _timers;    // Array
    ...
};
  
struct __CFRunLoop {
    CFMutableSetRef _commonModes;     // Set
    CFMutableSetRef _commonModeItems; // Set
    CFRunLoopModeRef _currentMode;    // Current Runloop Mode
    CFMutableSetRef _modes;           // Set
    ...
};
```
这里有个概念叫`CommonModes`：一个Mode可以将自己标记为`Common`属性（通过将其ModeName添加到RunLoop的 "CommmonModes"中）。每当RunLoop的内容发生变化时，RunLoop都会自动将_CommonModeItems里的 Source/Observer/Timer同步到具有'Common'标记的所有Mode里。

应用场景:主线程的RunLoop里有两个预置的Mode:KCFRunLoopDefaultMode和UITrackingRunLoopMode。这两个Mode都已经被标记为"Common"属性。DefaultMode是App平时所处的状态，TrackingRunLoopMode是追踪ScrollView滑动时的状态。当你创建一个Timer并加到DefaultMode时，TImer会得到重复回调，但此时滑动一个TableView时，RunLoop会将mode切换到 TrackingRunLoopMode,这时Timer就不会被回调，并且也不会影响到滑动操作.

有时你需要一个Timer,在两个Mode中都能得到回调，一种办法就是将这个Timer分别加入这两个Mode.还有一种方式，就是将TImer加入到顶层RunLoop的"commonModeItems"中，“commonMOdeItems”被RunLoop自动更新到所有具有"Common"属性的Mode里去.

CFRunLoop对外暴露的管理Mode接口只有下面2个:

```
CFRunLoopAddCommonMode(CFRunLoopRef runloop, CFStringRef modeName);
CFRunLoopRunInMode(CFStringRef modeName, ...);
```

Mode暴露的管理Mode Item的有下面几个:


```
CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
```
##RunLoop的内部逻辑
根据苹果官方文档的说明，RunLoop内部逻辑大致如下:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/1432798974517485.png)

实际上，RunLoop就是这样一个函数，其内部是一个 do-while循环，当你调用 CFRunLoopRun()时，线程就会一直停留在这个循环里，知道超时或被手动停止，该函数才会返回.

##苹果用RunLoop实现的功能


首先我们可以先看一下App启动后RunLoop的状态:

```
CFRunLoop {
    current mode = kCFRunLoopDefaultMode
    common modes = {
        UITrackingRunLoopMode
        kCFRunLoopDefaultMode
    }
  
    common mode items = {
  
        // source0 (manual)
        CFRunLoopSource {order =-1, {
            callout = _UIApplicationHandleEventQueue}}
        CFRunLoopSource {order =-1, {
            callout = PurpleEventSignalCallback }}
        CFRunLoopSource {order = 0, {
            callout = FBSSerialQueueRunLoopSourceHandler}}
  
        // source1 (mach port)
        CFRunLoopSource {order = 0,  {port = 17923}}
        CFRunLoopSource {order = 0,  {port = 12039}}
        CFRunLoopSource {order = 0,  {port = 16647}}
        CFRunLoopSource {order =-1, {
            callout = PurpleEventCallback}}
        CFRunLoopSource {order = 0, {port = 2407,
            callout = _ZL20notify_port_callbackP12__CFMachPortPvlS1_}}
        CFRunLoopSource {order = 0, {port = 1c03,
            callout = __IOHIDEventSystemClientAvailabilityCallback}}
        CFRunLoopSource {order = 0, {port = 1b03,
            callout = __IOHIDEventSystemClientQueueCallback}}
        CFRunLoopSource {order = 1, {port = 1903,
            callout = __IOMIGMachPortPortCallback}}
  
        // Ovserver
        CFRunLoopObserver {order = -2147483647, activities = 0x1, // Entry
            callout = _wrapRunLoopWithAutoreleasePoolHandler}
        CFRunLoopObserver {order = 0, activities = 0x20,          // BeforeWaiting
            callout = _UIGestureRecognizerUpdateObserver}
        CFRunLoopObserver {order = 1999000, activities = 0xa0,    // BeforeWaiting | Exit
            callout = _afterCACommitHandler}
        CFRunLoopObserver {order = 2000000, activities = 0xa0,    // BeforeWaiting | Exit
            callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv}
        CFRunLoopObserver {order = 2147483647, activities = 0xa0, // BeforeWaiting | Exit
            callout = _wrapRunLoopWithAutoreleasePoolHandler}
  
        // Timer
        CFRunLoopTimer {firing = No, interval = 3.1536e+09, tolerance = 0,
            next fire date = 453098071 (-4421.76019 @ 96223387169499),
            callout = _ZN2CAL14timer_callbackEP16__CFRunLoopTimerPv (QuartzCore.framework)}
    },
  
    modes ＝ {
        CFRunLoopMode  {
            sources0 =  { /* same as 'common mode items' */ },
            sources1 =  { /* same as 'common mode items' */ },
            observers = { /* same as 'common mode items' */ },
            timers =    { /* same as 'common mode items' */ },
        },
  
        CFRunLoopMode  {
            sources0 =  { /* same as 'common mode items' */ },
            sources1 =  { /* same as 'common mode items' */ },
            observers = { /* same as 'common mode items' */ },
            timers =    { /* same as 'common mode items' */ },
        },
  
        CFRunLoopMode  {
            sources0 = {
                CFRunLoopSource {order = 0, {
                    callout = FBSSerialQueueRunLoopSourceHandler}}
            },
            sources1 = (null),
            observers = {
                CFRunLoopObserver >{activities = 0xa0, order = 2000000,
                    callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv}
            )},
            timers = (null),
        },
  
        CFRunLoopMode  {
            sources0 = {
                CFRunLoopSource {order = -1, {
                    callout = PurpleEventSignalCallback}}
            },
            sources1 = {
                CFRunLoopSource {order = -1, {
                    callout = PurpleEventCallback}}
            },
            observers = (null),
            timers = (null),
        },
         
        CFRunLoopMode  {
            sources0 = (null),
            sources1 = (null),
            observers = (null),
            timers = (null),
        }
    }
}

```
可以看到，系统默认注册了5个Mode;

1. kcfRunLoopDefaultMode:App默认的Mode,通常主线程是在这个Mode下运行的
2. UITrackingRunLoopMode:界面跟踪Mode,用于ScrollView追踪触摸滑动，保证界面滑动时不受其他Mode影响
3. UIInitializationRunLoopMode:在刚启动App时进入的第一个Mode,启动完成后不再使用
4. CGEventReceiveRunLoopMode:接受系统事件的内部Mode,通常用不到
5. KcfRunLoopCommonModes:这是一个占位的Mode,没有实际作用

当RunLoop进行回调时，一般都是通过一个很长的函数调用出去(call out),当你在你的代码中断点调试时，通常能在调用栈上看到这些函数。下面就是这几个函数的整理版本，如果你在你的调用栈中看到这些长函数名，在这里查找一下就能定位到具体的调用地点了:

```
{
    /// 1. 通知Observers，即将进入RunLoop
    /// 此处有Observer会创建AutoreleasePool: _objc_autoreleasePoolPush();
    __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopEntry);
    do {
  
        /// 2. 通知 Observers: 即将触发 Timer 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeTimers);
        /// 3. 通知 Observers: 即将触发 Source (非基于port的,Source0) 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeSources);
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
  
        /// 4. 触发 Source0 (非基于port的) 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__(source0);
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
  
        /// 6. 通知Observers，即将进入休眠
        /// 此处有Observer释放并新建AutoreleasePool: _objc_autoreleasePoolPop(); _objc_autoreleasePoolPush();
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeWaiting);
  
        /// 7. sleep to wait msg.
        mach_msg() -> mach_msg_trap();
         
  
        /// 8. 通知Observers，线程被唤醒
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopAfterWaiting);
  
        /// 9. 如果是被Timer唤醒的，回调Timer
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__(timer);
  
        /// 9. 如果是被dispatch唤醒的，执行所有调用 dispatch_async 等方法放入main queue 的 block
        __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(dispatched_block);
  
        /// 9. 如果如果Runloop是被 Source1 (基于port的) 的事件唤醒了，处理这个事件
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__(source1);
  
  
    } while (...);
  
    /// 10. 通知Observers，即将退出RunLoop
    /// 此处有Observer释放AutoreleasePool: _objc_autoreleasePoolPop();
    __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopExit);
}
```

###AutoReleasePool
App启动后，苹果在主线程RunLoop里注册了两个Observer,其回调都是
`_wrapRunLoopWithAutoreleasePoolHandler()。`

第一个Observer监视的事件是进入Loop,其回调内都会调用` _objc_autoreleasePoolPush()`,创建自动释放池。

第二个Observer监视了两个事件：BeforeWaiting时调用
_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush()释放旧的池并创建新池；Exit(即将推出Loop)时调用`_objc_autoreleasePoolPop()`来释放自动释放池。


在主线程执行的代码，通常都是写在事件回调，Timer回调内的，这些回调会被RunLoop创建好的AutoreleasePool环绕着，所以不会出现内存泄露，开发者也不必显示创建Pool了。


###手势识别
当上面的`_UIApplicationHandleEventQueue() `识别了一个手势时，其首先会调用Cancel将当前的 `touchesBegin/Move/End `系列回调打断.随后系统将对应的`UIGestureRecognizer `标记为待处理。

苹果注册了一个Observer检测BeforeWaiting (Loop即将进入休眠) 事件,这个Observer的回调函数是 `_UIGestureRecognizerUpdateObserver()`,其内部会获取所有刚被标记为待处理的 GestureRecognizer,并执行GestureRecognizer的回调

###界面更新
当在操作UI时，比如改变了Frame,更新了UIview/CaLayer的层次时，或者手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后,这个UIview/CALayer就会标记为待处理，并被提交到一个全局的容器中。

苹果注册了一个Observer监听BeforeWaiting(即将进入睡眠)和Exit(即将退出Loop)事件，回调去执行一个很长的函数:

###定时器
NStimer其实就是`CFRunLoopTimerRef `,他们之间是toll-free bridged的，一个NStimer注册到RunLoop后，RunLoop会为其重复的时间点注册号通知，例如10:00,11:00,12:00,这几个时间点，

如果某个时间点被错过了，例如执行一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行，就好比等公交，如果10:10时，我忙着玩手机错过了，那我只能等10：20的那趟公交了。


CADisplayLink是一个和屏幕刷新率一致的定时器，如果在两次屏幕刷新之间执行了一个很长的任务，那其中就会有一帧被跳过去，造成界面卡顿的感觉，在快速滚动TableView时，即使一帧的卡顿也会让用户有所感觉.FaceBook开源的 [AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit.git)就是为了解决界面卡顿的问题，其内部也用到了RunLoop。

###performSelecter
当调用NSobject的`performSelecter:afterDelay:`后，实际上其内部会创建一个Timer并添加到当前线程的RunLoop中。所以如果当前线程没有RunLoop,则这个方法会失效.

当调用 `performSelector:onThread:`时，实际上其会创建一个TImer加到对应的线程中，同样滴，如果对应线程没有RunLoop该方法也会失效.

###关于GCD
实际上RunLoop底层也会用到GCD的东西，比如RUnLoop是用dispatch_source_t 实现的Timer.但同时GCD提供的某些接口也用了RUnLoop，比如dispatch_async().

当调用`dispatch_async(dispatch_get_main_queue(), block) `时，libDispatch会想主线程的RunLoop发送消息，RunLoop会被唤醒，并从消息中去的这个block,并在回调.

###关于网络请求
ios中，关于网络请求的接口自下而上如下几层:

```
CFSocket
CFNetwork       ->ASIHttpRequest
NSURLConnection ->AFNetworking
NSURLSession    ->AFNetworking2, Alamofire
```

* CFSocket是最底层的接口，值负责socket通信
* CGNetwork是基于cfSocket等接口的上层封装
* NSUrlConnection是基于CFNetwork的更高层的封装，提供面向对象的接口，AFNetworking工作于这一层
* NSURlSession是ios7中新增的接口，表面和NSUrlConnection并列的，但底层仍然用到了NSURLConnection 的部分功能，AFNetworking2 和 Alamofire 工作于这一层。
*

下面主要介绍NSURlConnection的工作过程.

通常使用NSURLconnection时，你会传入一个Delegate,当你调用 [connection start] 后，这个delegate就会不停的收到事件回调。实际上，start这个函数的内部会获取 CurrentRunLoop,然后在其中的DefaultMode添加了4个Source0, CFMultiplexerSource是负责各种Delegate回调的, CFHTTPCookieStorage是处理各种Cookie的。

当开始网络传输时，我们可以看到NSURLCOnnenction创建了两个新线程:
com.apple.NSURLConnectionLoader 和 com.apple.CFSocket.private.其中，CFSocket线程是处理底层socket连接的。NSUrlConnnectionLoader这个线程内部会使用RunLoop来接收底层socket的事件，并通过该之前添加的Source0通知上层的delegate。
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/1432799200369980.png)

NSURLConnectionLoader 中的 RunLoop 通过一些基于 mach port 的 Source 接收来自底层 CFSocket 的通知。当收到通知后，其会在合适的时机向 CFMultiplexerSource 等 Source0 发送通知，同时唤醒 Delegate 线程的 RunLoop 来让其处理这些通知。CFMultiplexerSource 会在 Delegate 线程的 RunLoop 对 Delegate 执行实际的回调


##RunLoop的实际应用举例
AFURLConnectionOperation 这个类是基于`NSURLConnection`构建的，其希望能在后台线程接收Delegate回调。为此，AFNetworking单独创建了一个线程，并在这个线程中启动了一个RunLoop:

```
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}
  
+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}
```
RunLoop启动前内部必须要有至少一个 Timer/Observer/Source,所以AFNetworking在[run start]之前放入了一个新的NSmachPort添加进入了。通常情况下，调用者需要持有NSMachPort (mach_port),并在外部线程通过这个Port发送消息到loop内；但此处添加port只是为了让RunLoop不至于退出，并没有实际的发送消息.


```
- (void)start {
    [self.lock lock];
    if ([self isCancelled]) {
        [self performSelector:@selector(cancelConnection) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    } else if ([self isReady]) {
        self.state = AFOperationExecutingState;
        [self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    }
    [self.lock unlock];
}
```

当需要在这个后台线程执行任务时，AFNetworking通过调用[NSObject performSelector:onThread:..] 将这个任务扔到了后台线程的RunLoop中。

##AsyncDisplayKit
AsyncDisplayKit 是 Facebook 推出的用于保持界面流畅性的框架，其原理大致如下:
UI线程中一旦出现繁重的任务就会导致界面卡顿，这类任务通常分为3类：排版，绘制，Ui对象操作.

排版通常包括计算视图的大小，计算文本的高度，等操作。

绘制一般有文本绘制，例如CoreText,图片绘制，例如预先解压，元素图形绘制等.

UI对象操作通常包括UIView/CaLayer等ui 对象的创建，设置属性和销毁.

其中前两类操作可以通过各种方法扔到后台线程中执行，而最后一类操作只能在主线程中完成，并且有时后面的操作需要一栏前面操作的结果。（例如UITextView创建时可能需要提前计算出文本的大小）.ASDK所做的，就是尽量将能放入到后台的任务放入到后台，不能则尽量推迟(例如视图的创建，属性的调整)

为此，ASDK创建了一个名为 `ASDisplayNode `的对象，并在内部封装了UiView/CaLayer，它具有和UIView/CALayer相似的属性，例如 frame,backgroundColor等。所有这些尚需经都可以放到后台线程更改，开发者可以只通过Node来操作器内部的UIVidw/CaLayer，这样就可以将排版和绘制放入到了后台线程，但是无论怎么操作，这些属性总是需要在某个时刻同步到主线程的 UIview/CaLayer中。


ASDK仿照 QuartzCore/UIKit框架的模式，实现了一套类似的界面更新机制:即在主线程的RunLoop中添加一个Observer,监听了 kCFRunLoopBeforeWaiting 和 kCFRunLoopExit 事件,在收到回调时，遍历所有之前放入队列等待处理的任务，然后一一执行。
