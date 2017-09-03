---
layout: post
title: "ReactiveSwift入门"
date: 2016-11-16 10:29:29 +0800
comments: true
categories: swift
---

## Signal
一个signal类型的实例，代表了一个有时序的并且可以被观察(类似订阅)的事件流。

信号通常被用来表示正在进行中的事件流，比如通知，用户输入等。用户（或者只要能造成事件的东西）产生的事件发送或者被接受，事件就被传递到信号上，并且被推送(push-Driven)到任何观察者哪里，并且所有观察者都是同时收到这些事件。

如果你想访问一系列的事件，就必须观察一个信号，观察一个信号并不会触发任何副作用，可以这样理解。信号是由生产者生产和推动的，消费者（观察者）是不会对事件的生命周期有任何影响。在观察一个信号时，发送了什么事件，只能对这个事件操作，因为信号是由时序的，不能随机的访问其他事件。

信号可以通过原函数去操作，比如filter,map,reduce,也可以同时操作多个信号如zip,这些原函数只在nextEvents生效（也就是对complete,failure等不生效）


在一个信号的生命周期里，可以发送无数次的NextEvents事件，直到他们被终结，类似compleye,Failed,InterRupper.终止事件没有数据值，所以他们必须被单独处理。

## Subscription
一个信号通常被用来表示正在进行中的事件流，有时候他们被叫做热信号，这意味着订阅者可以错过一些在它订阅前发送的事件。订阅一个信号不会触发任何副作用。

```

3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
scopedExample("Subscription") {
    // Signal.pipe is a way to manually control a signal. the returned observer can be used to send values to the signal
    let (signal, observer) = Signal<int, nonerror>.pipe()
    let subscriber1 = Observer<int, nonerror>(next: { print("Subscriber 1 received \($0)") })
    let subscriber2 = Observer<int, nonerror>(next: { print("Subscriber 2 received \($0)") })
    print("Subscriber 1 subscribes to the signal")
    print("\(observer)")
    signal.observe(subscriber1)
    print("Send value `10` on the signal")
    // subscriber1 will receive the value
    observer.sendNext(10)
    print("Subscriber 2 subscribes to the signal")
    // Notice how nothing happens at this moment, i.e. subscriber2 does not receive the previously sent value
    signal.observe(subscriber2)
    print("Send value `20` on the signal")
    // Notice that now, subscriber1 and subscriber2 will receive the value
    observer.sendNext(20)
}
--- Subscription ---
Subscriber 1 subscribes to the signal
Observer<int, nonerror>(action: (Function))
Send value `10` on the signal
Subscriber 1 received 10
Subscriber 2 subscribes to the signal
Send value `20` on the signal
Subscriber 1 received 20
Subscriber 2 received 20</int, nonerror></int, nonerror></int, nonerror></int, nonerror>
```
因为Swift有泛型的存在，这样的话我们可以把Signal当做任何数据类型的容器，而不是像OC中利用上帝类型Id，更加方便传递数据

首先我们通过Signal.pipe()创建了一个信号和一个观察者。

奇怪的是，在RACOC部分中，我们很少主动创建观察者，我们通常直接订阅信号就可以。

在Swift中，通过pipe创建的信号是个热信号，类似于OC中的RACSubject系列，在RACSubject继承自RACSiganl又继承RACStream，RACStream是一个Monad,它可以代表数据和数据的一系列的操作如map,flatterMap,bind


RACSubject又遵守了RACSubscriber协议，这个协议定义了可以发送数据的操作。

所以RACSubject即是一个信号，又是一个观察者。

在Swift部分的实现中，Signal并没有实现发送数据的方法。所以它需要一个内部的Observer去发送数据。所以它被pipe直接返回。

在外部我们需要自己实例化一个Observer观察者。去订阅事件。

可能在你查看Pipe的实现的时候并不好理解。把尾随闭包补全相对好理解点。


做个总结：

* RACOC中：RACSubject = RACSignal + RACSubscriper，在订阅的时候，订阅者被放在了RACSubject内部存放，我们只需要去关注订阅的block实现即可。
* RACSwift中:Signal 仅仅就是一个信号，所以需要一个内部观察者去充当发送数据的工具。外部的订阅需要自己手动实例观察者
* 热信号：由于pipe方法返回的是热信号，所以一个订阅者会错过在订阅之前发送的事件
*

### empty
空信号直接发送一个interrupted事件

```
scopedExample("`empty`") {
    let emptySignal = Signal<int, nonerror>.empty
    let observer = Observer<int, nonerror>(
    failed: { _ in print("error not called") },
    completed: { print("completed not called") },
    interrupted: { print("interrupted called") },
    next: { _ in print("next not called") }
    )
    emptySignal.observe(observer)
}
--- `empty` ---
interrupted called</int, nonerror></int, nonerror>
```


### Never
一个Never信号不会发送任何事件

```
scopedExample("`never`") {
    let neverSignal = Signal<int, noerror>.never
    let observer = Observer<int, noerror>(
        failed: { _ in print("error not called") },
        completed: { print("completed not called") },
        interrupted: { print("interrupted not called") },
        next: { _ in print("next not called") }
    )
    neverSignal.observe(observer)
}
--- `never` ---</int, noerror></int, noerror>
```

### uniqueValues唯一值

仅从集合中发送一次相同事件--类似与arrayQueue变成了Setqueue

注意：这会造成被发送的值被保留下来，用于以后发送的时候来检查是否重复，你可以编写一个函数来过滤重复值，这样可以减少内存消耗。

```

2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
scopedExample("`uniqueValues`") {
    let (signal, observer) = Signal<int, noerror>.pipe()
    let subscriber = Observer<int, noerror>(next: { print("Subscriber received \($0)") } )
    let uniqueSignal = signal.uniqueValues()
    uniqueSignal.observe(subscriber)
    observer.sendNext(1)
    observer.sendNext(2)
    observer.sendNext(3)
    observer.sendNext(4)
    observer.sendNext(3)
    observer.sendNext(3)
    observer.sendNext(5)
}
--- `uniqueValues` ---
Subscriber received 1
Subscriber received 2
Subscriber received 3
Subscriber received 4
Subscriber received 5</int, noerror></int, noerror>
```

### map

把每一个发送的值转换成新的值

```
scopedExample("`map`") {
    let (signal, observer) = Signal<int, noerror>.pipe()
    let subscriber = Observer<int, noerror>(next: { print("Subscriber received \($0)") } )
    let mappedSignal = signal.map { $0 * 2 }
    mappedSignal.observe(subscriber)
    print("Send value `10` on the signal")
    observer.sendNext(10)
}
--- `map` ---
Send value `10` on the signal
Subscriber received 20</int, noerror></int, noerror>
```


### mapError
把收到的error值变成新的error值

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
scopedExample("`mapError`") {
        let userInfo = [NSLocalizedDescriptionKey: "??"]
        let code = error.code + 10000
        let mappedError = NSError(domain: "com.reactivecocoa.errordomain", code: code, userInfo: userInfo)
    let (signal, observer) = Signal<int, nserror>.pipe()
    let subscriber = Observer<int, nserror>(failed: { print("Subscriber received error: \($0)") } )
    let mappedErrorSignal = signal.mapError { (error:NSError) -> NSError in
        return mappedError
    }
    mappedErrorSignal.observe(subscriber)
    print("Send error `NSError(domain: \"com.reactivecocoa.errordomain\", code: 4815, userInfo: nil)` on the signal")
    observer.sendFailed(NSError(domain: "com.reactivecocoa.errordomain", code: 4815, userInfo: nil))
}
--- `mapError` ---
Send error `NSError(domain: "com.reactivecocoa.errordomain", code: 4815, userInfo: nil)` on the signal
Subscriber received error: Error Domain=com.reactivecocoa.errordomain Code=14815 "??" UserInfo={NSLocalizedDescription=??}</int, nserror></int, nserror>
```

### filter

用于过滤一些值

```
scopedExample("`filter`") {
    let (signal, observer) = Signal<int, noerror>.pipe()
    let subscriber = Observer<int, noerror>(next: { print("Subscriber received \($0)") } )
    // subscriber will only receive events with values greater than 12
    let filteredSignal = signal.filter { $0 > 12 ? true : false }
    filteredSignal.observe(subscriber)
    observer.sendNext(10)
    observer.sendNext(11)
    observer.sendNext(12)
    observer.sendNext(13)
    observer.sendNext(14)
}
--- `filter` ---
Subscriber received 13
Subscriber received 14</int, noerror></int, noerror>
```

### ignoreNil

在发送的值为可选类型中：如果有值，把值解包，如果是nil丢弃掉

```
scopedExample("`ignoreNil`") {
    let (signal, observer) = Signal<int?, noerror>.pipe()
    // note that the signal is of type `Int?` and observer is of type `Int`, given we're unwrapping
    // non-`nil` values
    let subscriber = Observer<int, noerror>(next: { print("Subscriber received \($0)") } )
    let ignoreNilSignal = signal.ignoreNil()
    ignoreNilSignal.observe(subscriber)
    observer.sendNext(1)
    observer.sendNext(nil)
    observer.sendNext(3)
}
--- `ignoreNil` ---
Subscriber received 1
Subscriber received 3</int, noerror></int?, noerror>
```

### take
take(num)只取前num此值得信号

```
scopedExample("`take`") {
    let (signal, observer) = Signal<int, noerror>.pipe()
    let subscriber = Observer<int, noerror>(next: { print("Subscriber received \($0)") } )
    let takeSignal = signal.take(2)
    takeSignal.observe(subscriber)
    observer.sendNext(1)
    observer.sendNext(2)
    observer.sendNext(3)
    observer.sendNext(4)
}
--- `take` ---
Subscriber received 1
Subscriber received 2</int, noerror></int, noerror>
```

### collect
在发送complete事件之后，观察者会收到一个由之前事件组成的数组

注意：如果在发送complete事件的时候，没有任何事件发送，观察者会收到一个空的数据

```
scopedExample("`collect`") {
    let (signal, observer) = Signal<int, noerror>.pipe()
    // note that the signal is of type `Int` and observer is of type `[Int]` given we're "collecting"
    // `Int` values for the lifetime of the signal
    let subscriber = Observer<[Int], NoError>(next: { print("Subscriber received \($0)") } )
    let collectSignal = signal.collect()
    collectSignal.observe(subscriber)
    observer.sendNext(1)
    observer.sendNext(2)
    observer.sendNext(3)
    observer.sendNext(4)
    observer.sendCompleted()
}
--- `collect` ---
Subscriber received [1, 2, 3, 4]</int, noerror>


```


## SignalProducer

一个信号发生器，是SignalProducer类型的实例，它可以创建信号(signals)并施加副作用（side effects）

信号发生器用来表示操作或者任务，比如网络请求，每一次对它的调用start()将会生成一个新的潜在操作，并允许调用者观察它的结果，还有一个startWithSignal()方法，会给出产生的信号，允许在必要的情况下监听多次。


根据start()方法的动作方式，被同一个信号发生器生成的信号可能会有不同的事件顺序或版本，甚至事件流完全不一样！和普通的信号不同，在观察者连接上之前，信号发生器不会开始工作（也就没有事件会生成），并且在每一个新的监听器连接上时其工作都会重新开始一个单独的工作流。


启动一个信号发生器会返回一个销毁器(disposable)，它可用来打断或取消被生成信号的工作

和信号一样，信号生成器可以通过map,filter等原函数操作，使用lift方法，所有信号的原函数可以被提升成为以信号生成器为对象的操作，除此以外，还有一些用来控制何时与如何启动信号生成器的原函数，比如times.


通过lift函数可以让热信号转变为冷信号。

### Subscription
一个信号生成器代表了一种可以在需要的时候才被启动的操作（不像signal是自启动的），这种信号是冷信号，在刚开始这个信号的状态也为冷（未激活），既然是冷信号，那么就意味着这一个观察者不会错过任何被信号生成器发出的值。

补充：像signal是创建的时候状态为cold(理解为未激活)，被订阅时状态为hot(理解为激活)

但是冷信号和热信号与状态为冷热是两个不同的概念，冷信号会带来副作用，热信号不会

```
scopedExample("Subscription") {
    let producer = SignalProducer<int, noerror> { observer, _ in
        print("New subscription, starting operation")
        observer.sendNext(1)
        observer.sendNext(2)
    }
    let subscriber1 = Observer<int, noerror>(next: { print("Subscriber 1 received \($0)") })
    let subscriber2 = Observer<int, noerror>(next: { print("Subscriber 2 received \($0)") })
    print("Subscriber 1 subscribes to producer")
    producer.start(subscriber1)
    print("Subscriber 2 subscribes to producer")
    // Notice, how the producer will start the work again
    producer.start(subscriber2)
}
--- Subscription ---
Subscriber 1 subscribes to producer
New subscription, starting operation
Subscriber 1 received 1
Subscriber 1 received 2
Subscriber 2 subscribes to producer
New subscription, starting operation
Subscriber 2 received 1
Subscriber 2 received 2</int, noerror></int, noerror></int, noerror>
```

像不像是RACDynamicSignal的创建方式，这不过不同与Sinal的是，这里的发送信号的观察者是在内部通过Signal.pipe()生成的，不需要外部创建。

SignalProduce是冷信号，任何一个订阅者/观察者都不会错过任何事件

start方类似Signal的 signal.observe()方法，只不过Signal的方法只有一个作用，就是关联一个观察者，而SignalProduce的start方法还多了一个激活信号的功能


### Empty

一个会立即调用complete事件的信号生成器

```
/*:
 ### `empty`
 A producer for a Signal that will immediately complete without sending
 any values.
 */
scopedExample("`empty`") {
    let emptyProducer = SignalProducer<int, noerror>.empty
    let observer = Observer<int, noerror>(
        failed: { _ in print("error not called") },
        completed: { print("completed called") },
        interrupted: { print("interrupted called") },
        next: { _ in print("next not called") }
    )
    emptyProducer.start(observer)
}
--- `empty` ---
completed called</int, noerror></int, noerror>
```

Signal调用的是interrup方法，暂时不知道为什么，可能是为了区分语义吧，Signal是有时序的，SignalProduce是没有时序的。


### Never

一个什么都不会发送的信号器

```
/*:
 ### `never`
 A producer for a Signal that never sends any events to its observers.
 */
scopedExample("`never`") {
    let neverProducer = SignalProducer<int, noerror>.never
    let observer = Observer<int, noerror>(
        failed: { _ in print("error not called") },
        completed: { print("completed not called") },
        next: { _ in print("next not called") }
    )
    neverProducer.start(observer)
}
--- `never` ---</int, noerror></int, noerror>
```


### buffer
创建一个事件队列可以回放已经发送的事件

当一个值被发送的时候，它会被放进缓冲区内，如果缓冲区已经溢出，就会丢弃旧的值

这些被缓存的值将会被保留，直到这个信号被终结，当一个信号启动的时候，如果队列里没有任何值，所有被发送的新值都会被自动转发到观察者那里，直到管着着收到一个终止事件。

当一个终止事件被发送到队列中，观察者不会再收到任何值，并且这个事件不会被计算buffer的缓冲区大小，所以没有缓存的值都会被丢弃。

```
scopedExample("`buffer`") {
    let (producer, observer) = SignalProducer<int, noerror>.buffer(2)
    observer.sendNext(1)
    observer.sendNext(2)
    observer.sendNext(3)
    var values: [Int] = []
    producer.start { event in
        switch event {
        case let .Next(value):
            values.append(value)
        default:
            break
        }
    }
    print(values)
    observer.sendNext(4)
    print(values)
    let subscriber = Observer<int,noerror>(next:{ bufferdValue in
        print("\(bufferdValue)")
    })
    producer.start(subscriber)
}
--- `buffer` ---
[2, 3]
[2, 3, 4]
3
4</int,noerror></int, noerror>
```

### startWithSignal
通过Producer返回一个Signal,当闭包调用时返回signal开始发送事件

闭包返回一个Disponsable，可以用来中断Signal或者完成

```
scopedExample("`startWithSignal`") {
    var started = false
    var value: Int?
    SignalProducer<int, noerror>(value: 42)
        .on(next: {
            value = $0
        })
        .startWithSignal { signal, disposable in
            print(signal)
            print(value) // nil
        }
    print(value)
}
--- `startWithSignal` ---
ReactiveCocoa.Signal<swift.int, result.noerror>
nil
Optional(42)</swift.int, result.noerror></int, noerror>
```

### startWithNext
通过信号生成器创建一个信号，并且给这个信号内部直接构建一个观察者，在指定的闭包中会直接订阅next事件。

返回一个Disposable,可以中断这个信号，中断之后这个闭包不会再被调用

```
scopedExample("`startWithNext`") {
    SignalProducer<int, noerror>(value: 42)
        .startWithNext { value in
            print(value)
        }
}
--- `startWithNext` ---
42</int, noerror>
```

这个订阅只能接受next事件

### startWithCompleted
同startWithNext，只不过只能接受complete事件

```
scopedExample("`startWithCompleted`") {
    SignalProducer<int, noerror>(value: 42)
        .startWithCompleted {
            print("completed called")
        }
}
--- `startWithCompleted` ---
completed called</int, noerror>
```

### startWithFailed
同startWithNext， 只不过只能接受Failer事件事件

```
scopedExample("`startWithFailed`") {
    SignalProducer<int, nserror>(error: NSError(domain: "example", code: 42, userInfo: nil))
        .startWithFailed { error in
            print(error)
        }
}
--- `startWithFailed` ---
Error Domain=example Code=42 "(null)"</int, nserror>
```

### startWithInterrupted
同startWithNext,只不过只能接受interrupted事件

```
scopedExample("`startWithInterrupted`") {
    let disposable = SignalProducer<int, noerror>.never
        .startWithInterrupted {
            print("interrupted called")
        }
    disposable.dispose()
}
--- `startWithInterrupted` ---
interrupted called</int, noerror>
```


### lift
这个相对难理解点，大致类似于RAC_OC部分中的bind函数，monad中bind函数

可以理解为所有的原函数都是通过lift去实现的，借用中间信号来实现一系列的信号变换

```
scopedExample("`lift`") {
    var counter = 0
    let transform: Signal<int, noerror> -> Signal<int, noerror> = { signal in
        counter = 42
        return signal
    }
    SignalProducer<int, noerror>(value: 0)
        .lift(transform)
        .startWithNext { _ in
            print(counter)
        }
}
--- `lift` ---
42</int, noerror></int, noerror></int, noerror>
```

### map
把每个值都转换为新的值

```
scopedExample("`map`") {
    SignalProducer<int, noerror>(value: 1)
        .map { $0 + 41 }
        .startWithNext { value in
            print(value)
        }
}
--- `map` ---
42</int, noerror>
```

### mapError

把收到的error转换为新的error

```
scopedExample("`mapError`") {
    SignalProducer<int, nserror>(error: NSError(domain: "mapError", code: 42, userInfo: nil))
        .mapError { Error.Example($0.description) }
        .startWithFailed { error in
            print(error)
        }
}
--- `mapError` ---
Example("Error Domain=mapError Code=42 \"(null)\"")</int, nserror>
```

### filter
过滤不符合条件的值

```
scopedExample("`filter`") {
    SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
        .filter { $0 > 3}
        .startWithNext { value in
            print(value)
        }
}
--- `filter` ---
4</int, noerror>
```

### take
take(num) 只取前几次的值

```
scopedExample("`take`") {
    SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
        .take(2)
        .startWithNext { value in
            print(value)
        }
}
--- `take` ---
1
2</int, noerror>
```

### observeOn
在指定调度器上分发事件

```
/*:
 ### `observeOn`
 Forwards all events onto the given scheduler, instead of whichever
 scheduler they originally arrived upon.
 */
scopedExample("`observeOn`") {
    let baseProducer = SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
    let completion = { print("is main thread? \(NSThread.currentThread().isMainThread)") }
    if #available(OSX 10.10, *) {
    baseProducer
        .observeOn(QueueScheduler(qos: QOS_CLASS_DEFAULT, name: "test"))
        .startWithCompleted(completion)
    }
    baseProducer
        .startWithCompleted(completion)
}
--- `observeOn` ---
is main thread? true</int, noerror>
```

## collect

在发送完成的时候将一系列的值聚合为一个数组

```
scopedExample("`collect()`") {
    SignalProducer<int, noerror> { observer, disposable in
            observer.sendNext(1)
            observer.sendNext(2)
            observer.sendNext(3)
            observer.sendNext(4)
            observer.sendCompleted()
        }
        .collect()
        .startWithNext { value in
            print(value)
        }
}
--- `collect()` ---
[1, 2, 3, 4]</int, noerror>
```

### collect(count:)
在发送数据的时候（不需要发送complete）的时候将一系列的值聚合为数组，数组的长度为count,如果有很多数据，将会返回多个数组

```
scopedExample("`collect(count:)`") {
    SignalProducer<int, noerror> { observer, disposable in
            observer.sendNext(1)
            observer.sendNext(2)
            observer.sendNext(3)
            observer.sendNext(4)
        observer.sendNext(5)
//            observer.sendCompleted()
        }
        .collect(count: 2)
        .startWithNext { value in
            print(value)
        }
}
--- `collect(count:)` ---
[1, 2]
[3, 4]</int, noerror>
```

### collect(predicate:) matching values inclusively
通过谓词将一系列的值聚合为一个数组，注意在发送complete时候，如果前面只剩下一个值，就不需要聚合（因为没有其它元素和最后一个元素聚合），直接返回一个只有一个元素的数组。如果没有数据则返回一个空数组

```
scopedExample("`collect(predicate:)` matching values inclusively") {
    SignalProducer<int, noerror> { observer, disposable in
//            observer.sendNext(1)
//            observer.sendNext(2)
//            observer.sendNext(3)
//            observer.sendNext(4)
            observer.sendCompleted()
        }
        .collect { values in values.reduce(0, combine: +) == 3 }
        .startWithNext { value in
            print(value)
        }
}
--- `collect(predicate:)` matching values inclusively ---
[]</int, noerror>
```

尝试打开注释看看会有什么结果

### collect(predicate:) matching values exclusively
和上一个不同的是，如果谓词成功就把之前的聚合在一起，可以理解为把成功的界限当做分隔符

```
scopedExample("`collect(predicate:)` matching values exclusively") {
    SignalProducer<int, noerror> { observer, disposable in
            observer.sendNext(1)
            observer.sendNext(2)
            observer.sendNext(3)
            observer.sendNext(4)
            observer.sendNext(5)
            observer.sendCompleted()
        }
        .collect { values, next in next == 3 || next == 5  }
        .startWithNext { value in
            print(value)
        }
}
--- `collect(predicate:)` matching values exclusively ---
[1, 2]
[3, 4] // 3满足了条件所以被分开
[5] // 5也是</int, noerror>
```


### combineLatestWith
将第一个信号生成器的values和被聚合信号生成器的最后一个值聚合为一个元组

新产生的信号生成器不会发送任何值，只是转发，任何一个原来的信号被中断，这个新的信号生成器也会中断

```
scopedExample("`combineLatestWith`") {
    let producer1 = SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
    let producer2 = SignalProducer<int, noerror>(values: [ 1, 2 ])
    producer1
        .combineLatestWith(producer2)
        .startWithNext { value in
            print("\(value)")
        }
}
--- `combineLatestWith` ---
(1, 2)
(2, 2)
(3, 2)
(4, 2)</int, noerror></int, noerror>
```


### skip

skip（num），跳过num此发送的事件

```
scopedExample("`skip`") {
    let producer1 = SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
    producer1
        .skip(2)
        .startWithNext { value in
            print(value)
        }
}
--- `skip` ---
3
4</int, noerror>
```

### materialize

将被发送的值(value)编程Event,允许他们被修改。还句话说，允许他们被修改，把一个值变成一个Monad

当收到一个complete或者Failure事件，这个新的信号生成器，会发送事件并且结束。当收到一个interrupted事件，这个新的信号生成器也会中断

```
scopedExample("`materialize`") {
    SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
        .materialize()
        .startWithNext { value in
            print(value)
        }
}
--- `materialize` ---
NEXT 1
NEXT 2
NEXT 3
NEXT 4
COMPLETED
// 注意 value  如果不做materialize就是Int类型
```

### sampleOn
当sampler（被操作的信号生成器）发送任何事件的时候，都转发原来信号生成器的最后一个值

如果当一个sampler启动时，当前的值没有被观察者，没有任何事情发生

新产生的信号生成器从源信号生成器哪里发送数据，如果两个信号生成器任何一个complete或者interrupt,新产生的都会中断

```
/*:
 ### `sampleOn`
 Forwards the latest value from `self` whenever `sampler` sends a Next
 event.
 If `sampler` fires before a value has been observed on `self`, nothing
 happens.
 Returns a producer that will send values from `self`, sampled (possibly
 multiple times) by `sampler`, then complete once both input producers have
 completed, or interrupt if either input producer is interrupted.
 */
scopedExample("`sampleOn`") {
    let baseProducer = SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
    let sampledOnProducer = SignalProducer<int, noerror>(values: [ 1, 2 ])
        .map { _ in () }
    let newProduce = baseProducer
        .sampleOn(sampledOnProducer)
      newProduce  .startWithNext { value in
            print(value)
        }
}
--- `sampleOn` ---
4
4</int, noerror></int, noerror>
sampler发送的2次值都被变换成baseProduce 的comlete前的最后一个值
```

### combinePrevious
向前合并，没法送一个值就结合历史发送数据的最后一个构造成一个新的元组返回。在第一个发送时由于没有历史数据，所以combinePrevious传递了一个默认值。当做第一次的合并。


```
scopedExample("`combinePrevious`") {
    SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
        .combinePrevious(42)
        .startWithNext { value in
            print("\(value)")
        }
}
--- `combinePrevious` ---
(42, 1) // 第一次没有历史记录默认值是42
(1, 2) // 第二次默认记录是1
(2, 3)
(3, 4)</int, noerror>
```

### scan

类似reduce,将值聚合为一个新的值，每次聚合都保留结果作为下次的默认值，首次需给出默认值

每次聚合都会发送这个值

```
scopedExample("`scan`") {
    SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
        .scan(0, +)
        .startWithNext { value in
            print(value)
        }
}
--- `scan` ---
1
3
6
10</int, noerror>
```

### reduce
和scan类似，区别为reduce只发送聚合后的值并且立即结束

```
scopedExample("`reduce`") {
    SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
        .reduce(0, +)
        .startWithNext { value in
            print(value)
    }
}
--- `reduce` ---
10</int, noerror>
```

### skipRepeats

跳过表达式里返回true的值，第一个值不会被跳过

```
scopedExample("`skipWhile`") {
    SignalProducer<int, noerror>(values: [ 3, 3, 3, 3, 1, 2, 3, 4 ])
        .skipWhile { $0 > 2 }
        .startWithNext { value in
            print(value)
        }
}
--- `skipRepeats` ---
1
2
3
1
2
4
1
```

### skipWhile

对每个值都去做判断，知道返回false,之前的值会被跳过

```
scopedExample("`skipWhile`") {
    SignalProducer<int, noerror>(values: [ 3, 3, 3, 3, 1, 2, 3, 4 ])
        .skipWhile { $0 > 2 }
        .startWithNext { value in
            print(value)
        }
}
--- `skipWhile` ---
1  // 到1 返回false  之前的值被忽略掉
2
3
4</int, noerror>
```

### takeUntilReplacement

在被替换的信号发生器发送信号之后，发送被替换的信号

```
scopedExample("`takeUntilReplacement`") {
    let (replacementSignal, incomingReplacementObserver) = Signal<int, noerror>.pipe()
    let baseProducer = SignalProducer<int, noerror> { incomingObserver, _ in
        incomingObserver.sendNext(1)
        incomingObserver.sendNext(2)
        incomingObserver.sendNext(3)
// 下面被替换的信号生成器发送了事件，之后就不再发送baseProducer的事件了
// 相当于被替换了
        incomingReplacementObserver.sendNext(42)
        incomingObserver.sendNext(4)
        incomingReplacementObserver.sendNext(42)
    }
    let producer = baseProducer.takeUntilReplacement(replacementSignal)
    producer.startWithNext { value in
        print(value)
    }
}
--- `takeUntilReplacement` ---
1
2
3
42
42</int, noerror></int, noerror>
```


### takeLast
在发送complete事件后只取count此数据

```
scopedExample("`takeLast`") {
    SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
        .takeLast(2)
        .startWithNext { value in
            print(value)
        }
}
只取了2次数据
--- `takeLast` ---
3
4</int, noerror>
```


### ignoreNil
如果发送的事件是可选类型，解包这些可选类型，并且丢弃nil值

```
scopedExample("`ignoreNil`") {
    SignalProducer<int?, noerror>(values: [ nil, 1, 2, nil, 3, 4, nil ])
        .ignoreNil()
        .startWithNext { value in
            print(value)
        }
}
--- `ignoreNil` ---
1
2
3
4</int?, noerror>
```

### zipWith
压缩信号生成器，只有再两个信号都有数据发送之后，新的信号生成器才会发送数据

新的数据被组合为元组

```
scopedExample("`zipWith`") {
    let baseProducer = SignalProducer(values: [ 1, 2, 3, 4 ])
    let zippedProducer = SignalProducer(values: [ 42, 43 ])
    baseProducer
        .zipWith(zippedProducer)
        .startWithNext { value in
            print("\(value)")
        }
}
--- `zipWith` ---
(1, 42)
(2, 43)
```

后面应为第二个没有数据了，所以不会再聚合了

### times
time(count)重复发送count数据，每次重复必须上次发送完成事件

```
scopedExample("`times`") {
    var counter = 0
    SignalProducer<(), NoError> { observer, disposable in
            counter += 1
            observer.sendCompleted()
        }
        .times(42)
        .start()
    print(counter)
}
--- `times` ---
42
```

### retry

如果收到失败事件重试retry(count)次

```
scopedExample("`retry`") {
    var tries = 0
    SignalProducer<int, nserror> { observer, disposable in
            if tries == 0 {
                tries += 1
                observer.sendFailed(NSError(domain: "retry", code: 0, userInfo: nil))
            } else {
                observer.sendNext(42)
                observer.sendCompleted()
            }
        }
        .retry(1)
        .startWithResult { result in
            print(result)
        }
}
--- `retry` ---
.Success(42)</int, nserror>
```

当第一个信号发送complete时，第二个信号被替换成信号发送线路上，如果有任何失败事件，后面的就替换失败。

第一个信号发送的所有事件都会被忽略

![1](http://cc.cocimg.com/api/uploads/20160726/1469505056118233.png)


### flatMap
将收到的每个事件都映射为新的Product,然后摊平，如果原来的producer发送失败，新产生也得立即失败

```
scopedExample("`flatMap(.Latest)`") {
    SignalProducer<int, noerror>(values: [ 1, 2, 3, 4 ])
        .flatMap(.Latest) { SignalProducer(value: $0 + 3) }
        .startWithNext { value in
            print(value)
        }
}
--- `flatMap(.Latest)` ---
4
5
6
7</int, noerror>
```

### flatMapError
把收到的failer事件映射为新的Producer,并且摊平它

```
scopedExample("`flatMapError`") {
    SignalProducer<int, nserror>(error: NSError(domain: "flatMapError", code: 42, userInfo: nil))
        .flatMapError { SignalProducer<int, noerror>(value: $0.code) }
        .startWithNext { value in
            print(value)
        }
}
--- `flatMapError` ---
```