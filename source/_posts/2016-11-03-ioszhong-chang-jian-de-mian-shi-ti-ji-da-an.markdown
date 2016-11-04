---
layout: post
title: "ios中常见的面试题及答案"
date: 2016-11-03 17:50:01 +0800
comments: true
categories: swift
---

## ios中深拷贝和浅拷贝

在ios开发中，经常涉及到深拷贝和浅拷贝的问题，针对深拷贝和浅拷贝，为了方便大家的理解，专门总结如下:
<!--more-->

* 理解1

浅拷贝是拷贝操作后，并没有进行真正的复制，而是另一个指针也指向了同一个地址。深拷贝，拷贝操作后，是真正的复制了一份，另一个指针指向了拷贝后的地址。如下图：A代表原有的指针，B代表拷贝的指针。（图一为浅拷贝，图二为深拷贝）

![1](http://img.blog.csdn.net/20141218004439540?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3NkbkFhcm9u/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![2](http://img.blog.csdn.net/20141218004554327?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3NkbkFhcm9u/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

从上图中可以看到，浅拷贝（浅复制）中如果其中A指针改变了所指向的地址的内容，那么B指针也指向了被修改的内容，如果有些地方用到B指针，即便A指向的内容发生变化，也不希望B收到影响，则需要用深拷贝，真正复制一份A指向的内容，B指向复制后的值，这样即使A指向的内容变化了，B也不会产生影响。好比：浅复制好比你的影子，你完蛋，你的影子也完蛋。深复制好比你和你的克隆人，你完蛋，你的克隆人依然活着。

* 理解2

深拷贝和浅拷贝的本质是地址相同，就是浅拷贝，地址不同就是深拷贝。

iOS开发过程中，大体上会区分为对象和容器两个概念，对象的copy是浅拷贝，mutableCopy是深拷贝。容器也参照如上方法，但是需要记住，容器的包含对象的拷贝,无论使用copy,还是mutableCopy都将是浅拷贝，想要实现对象的深拷贝，必须自己提供拷贝的方法。

* 理解3

```
 NSArray *array=[NSArray arrayWithObjects:@"one",@"two", nil];       
 NSMutableArray *array1=[array copy];        
[array1 addObject:@"three"];  
```

//这段代码是错误的，array1通过copy进行的是浅拷贝，即并没有真正复制array，而是也指向了array,此时array是不可变数组，无法进行新数据的添加

```
NSArray *array=[NSArray arrayWithObjects:@"one",@"two", nil];       
NSMutableArray *array2=[array mutableCopy];        
[array2 addObject:@"three"];  
```
这段代码是正确的，array2通过mutableCopy进行的是深拷贝，即把array真正复制了一份，并且复制后，变味了NSMutableArray,此时array2是可变数组，可以添加数据

>
>注意点:*(1)* 当使用mutableCopy时，不管源对象是否可变，副本是可变的，并且实现真正意义上的拷贝。当我们使用copy一个可变对象时，副本对象是不可变的。
>
>*(2)*要想实现对象的自定义拷贝，必须实现NSCopying,NSMutableCopying协议，实现该协议的copyWithZone方法和mutableCopyWithZone方法。深拷贝和浅拷贝的区别就在于copyWithZone方法的实现。


## NSString属性什么时候用copy,什么时候用Strong?

我们定义一个类，并且为其声明两个字符串属性，如下所示：

```
@interface TestStringClass ()
@property (nonatomic, strong) NSString *strongString;
@property (nonatomic, copy) NSString *copyedString;
@end
```

上面的代码声明了两个字符串属性，其中一个内存特性是strong,一个是copy.下面我们来看看它们的区别。

首先，我们用一个不可变字符串来为这两个属性赋值,

```
- (void)test {
    NSString *string = [NSString stringWithFormat:@"abc"];
    self.strongString = string;
    self.copyedString = string;
    NSLog(@"origin string: %p, %p", string, &string);
    NSLog(@"strong string: %p, %p", _strongString, &_strongString);
    NSLog(@"copy string: %p, %p", _copyedString, &_copyedString);
}
```

输出结果为:

```
origin string: 0x7fe441592e20, 0x7fff57519a48
strong string: 0x7fe441592e20, 0x7fe44159e1f8
copy string: 0x7fe441592e20, 0x7fe44159e200
```
我们可以看到，这种情况下，不管是strong还是copy属性的对象，其指向的地址都是同一个，即为string执行的地址，如果我们换做MRC环境，打印string的引用计数的话，会看到其引用计数是3,即String操作和copy做做都使原字符串对象的引用计数值+1.

接下来，我们把string由不可变改为可变对象，看看会是什么结果，即将下面这一句

```
NSString *string = [NSString stringWithFormat:@"abc"];
```
改成:

```
NSMutableString *string = [NSMutableString stringWithFormat:@"abc"];
```

其输出结果为:

```
origin string: 0x7ff5f2e33c90, 0x7fff59937a48
strong string: 0x7ff5f2e33c90, 0x7ff5f2e2aec8
copy string: 0x7ff5f2e2aee0, 0x7ff5f2e2aed0
```

可以发现，此时copy属性字符串已不再指向string字符串对象，而是深拷贝了string字符串，并让_copyedString对象指向这个字符串，在MRC环境下，打印两者的引用计数，可以看到string对象的引用计数是2，而_copyedString的引用计数是1

此时，我们如果去修改string字符串的话，可以看到：因为_strongString和string都是指向同一个对象，所以_strongString的值会跟随者改变（需要注意的是，此时 _strongString 的类型实际上是NSmutableString,而不是NSString）而_copyedString指向的是另一个对象，所以并不会改变。

### 结论
由于NSMutableString是NSString的子类，所以一个NSString指针可以指向NSMutableString对象，让我们的StrongString指针指向一个可变字符串是OK的。

而上面的例子可以看得出，当源字符串是NSString时，由于字符串是不可变的，所以，不管是Strong还是copy属性的对象，都是指向源对象，copy操作只是做了次浅拷贝。

当源字符串是NSMUtableSring时，strong属性只是增加了源字符串的引用计数，而copy属性则是对源字符串做了次深拷贝，产生了一个新的对象，且copy属性对象指向这个新的对象。另外需要注意的是，这个copy属性对象的类型始终是NSString,而不是NSMutableString,因此其实不可变的。

这里还有一个性能问题，即在源字符串是NSMutableString,strong是单纯的增加对象的引用计数，而copy操作是智行了一次深拷贝，所以性能上会有所差异，而如果源字符串是NSString时，则没有这个问题。

## OC的理解和特性


* OC 作为一门面向对象的语言，自然具有面向对象的语言特性：封装，继承，多台，它既有静态语言的特性（如C++），又有动态语言的效率（动态绑定，动态加载）。总体来讲，OC确实是一门不错的编程语言
* OC具有相当多的动态特性，表现为三个方面：动态类型（Dynamic typing）,动态绑定(Dynamic binding)和动态加载(Dynamic loading).动态-必须运行时（run time）才会做的事情

* 动态类型：即运行时再决定对象的类型，这种动态特性在日常的应用中非常常见，简单说就是id类型，事实上，由于静态类型的固定性和可预知性，从而使用的更加广泛，静态类型是强类型，而动态类型属于弱类型，运行时决定接受者
* 动态绑定：基于动态类型，在某个实例对象被确定后，其类型便被确定了，该对象对对那个的属性和响应消息也被完全确定。
* 动态加载：根据需求加载所需要的资源，最基本就是不同机型的适配，例如，在Retain设备上加载@2x的图片，而在老一些的普通苹果设备上加载原图，让程序在运行时添加代码模块以及其他资源，用户可根据需要加载一些可执行代码和资源，而不是在启动时就加载所有组件，可执行代码可以含有和程序运行时整合的新类

## 简述内存管理的基本原则
* 之前：OC内存管理遵循“谁创建，谁释放，谁引用，谁管理”的机制，当创建活引用一个对象的时候，需要向它发送alloc,copy,retain消息，当释放该对象时需要发送release消息，当对象引用计数为0时，系统将释放该对象，这是OC的手动管理机制(MRC)
* 目前：ios5之后引用自动管理机制-自动引用计数（ARC），管理机制和手动机制一样，只是不再需要调用retain,release,autorelease,它编译时的特性，当你使用arc时，在适当位置插入release和 autorelease;它引用strong和weak关键字，strong修饰的指针变量指向对象时，当指针指向新值或者指针不复存在，相关联的对象就会自动释放，而weak修饰的指针比那两指向对象，当对象的拥有者指向新值或者不存在时weak修饰的指针会自动设置为nil
* 如果使用alloc,copy(mutableCopy)或者retain一个对象时，你就有义务向它发送一条release或者autorelease消息，其它方法创建的对象，不需要由你来管理内存。
* 向一个对象发送一条autorelease消息，这个对象并不会立即销毁，而是将这个对象放入了自动释放池，待池子释放时，它会向池子中每个对象发送一条release消息，以此来释放对象
* 向一个对象发送release消息，并不意味着这个对象被销毁了，而是当这个对象的引用计数为0时，系统才会调用dealloc方法，释放该对象和对象本身所拥有的实例。

## 其它注意事项
* 如果一个对象有一个_Strong类型的指针指向着，找个对象就不会被释放。如果一个指针指向超出了它的作用域，就会被指向nil.如果一个指针被指向nil,那么它原来指向的对象就被释放了，当一个视图控制器被释放时，它内部的全部指针会被指向nil."不管是全局变量还是局部变量用_Strong描述就行"
* 局部变量：出了作用域，指针会被设置为nil
* 方法内部创建对象，外部使用需要添加_autorelease
* 连线的时候，用_weak描述
* 代理使用unsafe_unretained就相当于assign
* block中为了避免循环引用问题，使用_weak描述
* 声明属性时，不要以new开头，如果非要以new开头命名属性的名字，需要自己定制get方法名，如

```
@property(getter=theString) NSString * newString;
```

* 如果要使用自动释放池，用@autoreleasepool{}
* ARC只能管理Foundation框架的变量，如果程序中把Foundation中的变量强制换成Core Foundation中的变量需要交换管理权
* 在非ARC工程中采用ARC去编译某些类:`-fobjc-arc`
* 在ARC工程下采用非ARC去比哪一某些类:`-fno-fobjc-arc`

## 如何理解MVC设计模式
mvc是一种架构模式，M表示Model，V表示视图View,C表示控制器Controller:
* Model负责存储，定义，操作数据
* View用来展示数据，和用户进行交互
* Controller是Model和View的协调者，Controller把MOdel中的数据拿过来给View用，Controller可以直接与Model和View进行通信，而View不能和Controller直接通信。View与Controller通信需要利用代理协议的方式，当有数据更新时，Model也要与Controller进行通信，这个时候要用Notification和KVO,这个方式就像一个广播一样，Model刚发送信号，Contrller设置坚挺接受信号，当有数据更新时就发信号给Controller,Model和View不能直接进行通信，这样会违背MVC设计模式

## 如何理解MVVM设计模式
* ViewModel层，就是View和Model层的粘合剂，他是一个放置用户输入验证逻辑，视图显示逻辑，发起网络请求和其它业务逻辑处理极好的地方，说白了，就是把原来ViewCOntroller层的业务逻辑和页面逻辑等剥离出来放到ViewModel层
* View层，就是ViewController层，它的任务就是从ViewModel层获取数据，然后显示

## Objective-C 中是否支持垃圾回收机制？
* OC是支持垃圾回收机制的,但是Apple的移动终端中，是不支持GC的，Mac桌面系统开发中是支持的。
* 移动端开发是支持ARC的，ARC是在ios5之后推出的新技术，它与GC机制是不同的，我们在编写代码时，不需要想对象发送release或者autorelease方法，也不可以调用dealloc方法，编译器会在合适的位置自动给用户生成release消息，ARC的特点是自动引用计数简化了内存管理的难度

## ARC下Assign和weak的区别

weak比assign对了一个功能就是当属性所指向的对象消失的时候（也就是内存引用计数为0）会自动赋值nil,这个再向weak修饰的属性发送消息就不会导致野指针操作crash.

在arc模式下编程时，指针变量一定要用weak修饰，只有基本数据类型和结构体需要用assign，例如delegate,一定要用weak修饰。

* 区别：如果用weak声明的变量在栈中就会自动清空，赋值为nil，如果用assign声明的变量在栈中可能不会自动赋值为nil,就会造成野指针错误

## 协议的基本概念和协议中方法默认什么类型

OC中的协议是一个方法列表。它的特点是可以被任何类使用（实现），但它并不是类，自身不会实现这样方法，而是有其它人来实现协议，经常用来实现委托对象，如果一个类采用了一个协议，那么它必须实现协议中必须需要实现的方法，在协议中的方法默认是必须实现的（@required）,添加关键字@optional,表明一旦采用该协议，这些可选的方法是可以不实现的

## 简述类目Category的有点和缺点

### 优点
* 不需要通过增加子类而增加现有类的行为或方法，且类目中的方法与原始类方法基本没有区别：
* 通过类目可以将庞大的一个类的方法进行划分，从而便于代码的日后的维护，更新及提高代码的阅读性

### 缺点
* 无法向类目中添加实例变量，如果需要添加实例变量，只能通过定义子类的方式
* 类目中的方法与原始类以及父类方法相比具有更高优先级，如果覆盖弗雷的方法，可能导致super消息的断裂，因此，最好不要覆盖原始类中的方法。


## 循环引用产生额原因，以及解决方法
* 产生原因：如下图所示，对象A和对象B相互引用了对方作为自己的成员变量，只有自己销毁的时候才能将成员变量的引用计数减去1.对象A的销毁依赖于对象B的销毁，同事对象B销毁也依赖于对象A的销毁，从而形成了循环引用，此时，即使外界没有任何指针访问它，它也无法释放。
![2](http://devstorepic.qiniudn.com/FvDA-QQdrUBpndLKOmJgy6-vqM0F)


对个对象之间依然会存在循环引用问题，形成一个环，在编程中，形成的环越大越不容易察觉，如下图所示：

![1](http://devstorepic.qiniudn.com/Fk4cV48OjN9tUl-lDiU_ap5WWGUl)

### 解决方法
* 事先知道存在循环引用的地方，在合理的位置主动断开一个引用，让对象回收
* 使用weak声明

## 键路径(keyPath),键值编码(KVC),键值观察(KVO)

### 键路径
* 在一个给定的实体中，同一个属性的所有值具有相同的数据类型
* 键-值编码技术用于进行这样的查找-它是一种间接访问对象属性的机制。键路径是一个由点做分隔符组成的字符串，用于指定一个连接在一起的对象性质序列。第一个键的性质是由先前的性质决定的，接下来每个键的值也是相对于前面的性质
* 键路径使您可以以独立于模型实现的方式指定相关对象的性质。通过键路径，您可以指定对象图中的一个任意深度的路径，使其指向相关对象的特定属性
### 键值编码KVC

* 键值编码是一种间接访问对象的属性使用字符串来标识属性，而不是通过调用存取的方法，直接或通过实例变量访问的机制，非对象类型的变量将被自动封装或者解封成对象，很多情况下会简化程序代码；
* KVC的缺点：一旦使用KVC，你的编译器无法检查出错误，即不会对设置的键，键路径进行错误检查，且执行效率要地域合成存取器方法和自定的setter和getter方法，因为使用KVC键值编码，它必须先解析字符串，然后在设置或者访问对象的实例变量
### 键值观察KVO
* 键值观察机制是一种能使的对象获取到其他对象属性变化的通知，极大的简化了代码
* 实现KVO键值观察模式，被观察的对象必须使用KVC键值编码来修改它的实例变量，这样才能被观察者观察到，因此，KVC是KVO的基础

### Demo
比如我自定义一个Button

```
[self addObserver:self forKeyPath:@"highlighted" options:0 context:nil]; #pragma mark KVO 
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context { 
     if ([keyPath isEqualToString:@"highlighted"] ) { 
      [self setNeedsDisplay]; } 
  }
```


对于系统是根据keypath去取的到相应的值发生改变，理论上来说是和KVC机制的道理是一样的。

### KVC机制通过key找到value的原理
* 当通过KVC调用对象时，比如：`[self valueForKey:@”someKey”]`,程序会自动视图通过下面几种不同的方式解析这个调用。
* 首先查找对象是否带有somekey这个方法，如果没找到，会继续查找对象是否带有somekey这个实例变量，如果还没有找到，程序会继续视图调用` -(id) valueForUndefinedKey:`这个方法，如果这个方法还是没有被实现的话，程序会抛出一个`NSUndefinedKeyException `错误
* 补充：KVC在查找方法的时候，不仅会超照somekey这个方法，还会查找getsomeKey这个方法，前面加一个get,或者_someKey以_getsomeKey这几种形式，同时，查找实例变量的时候也会不仅仅查找somekey这个变量，也会查找_someKey这个变量是否存在
* 设计`valueForUndefinedKey `方法的主要目的是当你使用-(id)valueForKey方法从对象中请求值时，对象能够在错误发生前，有最后的机会响应这个请求。

## 在Objective-C中如何实现KVO
* 注册观察者（注意：观察者和被观察者不会被保留也不会被释放）

```
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath 
options:(NSKeyValueObservingOptions)options 
context:(void *)context;

- (void)observeValueForKeyPath:(NSString *)keyPath 
ofObject:(id)object change:(NSDictionary *)change   context:(void *)context;

- (void)removeObserver:(NSObject *)observer 
forKeyPath:(NSString *)keyPath;

```

* KVO 中谁要监听谁注册，然后响应进行处理，使得观察者与被观察者完全解耦。KVO只检测类中的属性，并且属性都是通过NSString来查找，编译器不会检错和补位，全部取决于自己

## 代理的作用
* 代理又叫委托，是一种设计模式，代理是对象与对象之间的通信交互，代理解除了对象之间的耦合性
* 改变或传递控制链，允许一个类在某些特定时刻通知到其他类，而不需要获取到哪些类的指针，可以减少框架的复杂度
* 另外一点，代理可以理解为java中回调监听机制的一种类似
* 代理的属性常常是assign的原因：防止循环引用，以至于对象无法得到正确的释放

## NSNotification、Block、Delegate和KVO的区别
* 代理是一种回调机制，且是一对一的关系，通知是一对多的关系，一个对向所有的观察者提供变更通知
* 效率：Delegate比NSNotification高
* Delegate和Block一般是一对一的通信
* Delegate需要定义协议方法，代理对象实现协议方法，并且需要建立代理关系才可以实现通信
* Block:Block更加简洁，不需要定义繁琐的协议方法，但通信事件比较多的话，建议使用Delegate;

## Objective-C中可修改和不可修改类型
* 可修改不可修改的集合类，就是可动态添加修改和不可动态添加修改
* 比如NSArray和NSMutableArray,前者在初始化后的内存控件就是固定不变的，而后者可以添加修改等，可以动态申请的内存空间

## 当我们调用一个静态方法时，需要对对象进行release吗？
不需要，静态方法（类方法）创建一个对象时，对象已被放入自动释放池。在自定释放池被释放时，很有可能被销毁

## 当我们释放我们的对象时，为什么需要调用[super dealloc]方法，它的位置又是如何的呢？
* 因为子类的某些实例是继承自父类的，因此需要调用`[super dealloc]`方法，来释放父类拥有的实例，其实也就是子类本身的，一般来说我们优先释放子类拥有的实例，最后释放父类所拥有的实例

## 对谓词的认识
* Cocoa中提供乐意一个`NSPredicate `类，该类主要用于指定过滤器的条件，每一个对象通过谓词进行筛选，判断条件是否匹配

## static，self，super关键字的作用
* 函数体内static变量的作用范围为该函数体，不同于auto变量，该变量的内存只被分配了一次，因此其值在下次调用时扔维持上次的值
* 在模块内的static全局变量可以被模块内所有函数访问，但不能被模块外其他函数访问
* 在模块内的static函数只可被这一模块内的其他函数调用，这个函数的使用范围被限制在声明
* 在类中的static成员变量属于整个类所拥有过，对类的所有对象只有一份拷贝
* self：当前消息的接受者
* super：向父类发送消息

## include与#import的区别、#import 与@class 的区别

* include和#import其效果相同，都是查询勒种定义的行为
* import不会引起交叉编译，确保头文件只会被导入一次
* @class的表明，只定义了类的名称，而具体类的行为是未知的，一般用于.h文件
* @class比#import编译效率更高
* 此外@class和#import的主要区别在于解决引用死锁的问题

## @public、@protected、@private @ fileprivate, open 它们的含义与作用

* @public:对象的实例变量的作用域在任意地方都可以被访问
* @protected:对象的实例变量作用域在本类和子类都可以被访问到
* @private：实例变量的作用域只能在本类中访问

###  fileprivate
在原有的swift中的private其实并不是真正的私有，如果一个变量定义为private,在同一个文件中的其他类依然是可以访问到的。这个场景在使用extension的时候很明显

```
class User {
    private var name = "private"
}

extension User{
    var accessPrivate: String {
        return name
    }
}
```
这样带来了两个问题：

* 当我们标记为private时，意思为真的私有还是文件内共享呢？
* 当我们如果意图为真正的私有时，必须保证这个类或者结构体在一个单独的文件内，否则可能同文件里其它的代码访问到

由此，在swift3中，新增加了一个`fileprivate`来显示表明，这个元素的访问权限为文件内私有，过去的private对应现在的fileprivate,现在private则是真正的私有，离开了这个类或者结构体的作用域外面就无法访问到

### open
open则是弥补public语义上的不足。
现在public有两层含义:

* 这个元素可以在其他作用域被访问
* 这个元素可以在其他作用域被继承或者ovrride

继承是一件危险的事情，尤其对于一个framework或者module的设计者而言，在自身的module内，类或者属性对于作者而言是清晰的，能否被继承或者ovrride都是可控制的。但是对于使用它的人，作者有时会希望传达出这个类或者属性不应该被继承或者修改，这个对应的就是final.

final的问题在于标记之后，在任何地方都不能被ovrride,而对于lib的设计者而言，希望得到的是在module内可以ovrride,在被import到其他地方后其他用户使用的时候不能被ovrride.

这就是`open`产生的初衷，通过open和public标记区别一个元素在其他module中是只能被访问还是可以被ovrride.

例子:

```
/// ModuleA:

// 这个类在ModuleA的范围外是不能被继承的，只能被访问
public class NonSubclassableParentClass {

    public func foo() {}

    // 这是错误的写法，因为class已经不能被继承，
    // 所以他的方法的访问权限不能大于类的访问权限
    open func bar() {}

    // final的含义保持不变
    public final func baz() {}
}

// 在ModuleA的范围外可以被继承
open class SubclassableParentClass {
    // 这个属性在ModuleA的范围外不能被override
    public var size : Int

    // 这个方法在ModuleA的范围外不能被override
    public func foo() {}

    // 这个方法在任何地方都可以被override
    open func bar() {}

    ///final的含义保持不变
    public final func baz() {}
}

/// final的含义保持不变
public final class FinalClass { }
/// ModuleB:

import ModuleA

// 这个写法是错误的，编译会失败
// 因为NonSubclassableParentClass类访问权限标记的是public，只能被访问不能被继承
class SubclassA : NonSubclassableParentClass { }

// 这样写法可以通过，因为SubclassableParentClass访问权限为 `open`.
class SubclassB : SubclassableParentClass {

    // 这样写也会编译失败
    // 因为这个方法在SubclassableParentClass 中的权限为public，不是`open'.
    override func foo() { }

    // 这个方法因为在SubclassableParentClass中标记为open，所以可以这样写
    // 这里不需要再声明为open，因为这个类是internal的
    override func bar() { }
}

open class SubclassC : SubclassableParentClass {
    // 这种写法会编译失败，因为这个类已经标记为open
    // 这个方法override是一个open的方法，则也需要表明访问权限
    override func bar() { } 
}

open class SubclassD : SubclassableParentClass {
    // 正确的写法，方法也需要标记为open
    open override func bar() { }    
}

open class SubclassE : SubclassableParentClass {
    // 也可以显式的指出这个方法不能在被override
    public final override func bar() { }    
}


```

现在的访问权限则依次为:open,public,internal,fileprivate,private.



## iOS开发中数据持久性有哪几种？
数据存储的核心都是写文件

* 属性列表：只有NSString,NSArray,NSdictionary,NSdata可以writeToFile;存储依旧是plist文件，plist文件可以存储7种数据类型：array,dictionary,string,bool,data,date,number
* 对象序列化（对象归档）:对象序列化通过序列化的形式，键值关系存储到本地，转化为二进制刘，通过runtime实现自动化归档/解档，实现NSCoding协议必须实现的两个方法:
1. 编码(对象序列化):把不能直接存储到plist文件中得到数据，转化为二进制数据，NSData,可以存储到本地
2. 解码:(对象反序列化)把二进制数据转化为本来的类型
3. SqlLite数据库：大量有规律的数据使用数据库
4. CoreData:通过管理对象进行增删改查操作。它不是一个数据库，不仅可以使用SqlLite数据库来保持数据，也可以使用其他方式来存储数据，如:XML

###*CoreData介绍*

* CoreData是面向对象的API，COreData是ios中非常重要的一项技术，几乎在所有编写的程序中，CoreData都作为数据存储的基础
* CoreData是苹果官方提供的一套框架，用来解决与对象声明周期管理，对象关系管理和持久化等方面相关的问题
* 大多数情况下，我们引用CoreData作为持久化数据的解决方法，并利用它作为持久护士数据映射为内存对象。提供的是对象-关系映射功能，也就是说，CoreData可以将Objective-C对象转为数据，保存到SQL中，然后将保存后的数据还原成OC对象

###*CoreData的特征:*

* 通过CoreData管理应用程序的数据模型，可以极大程度减少需要编写的代码数量
* 将对象数据存储在SQLite数据已获得性能优化
* 提供NSFetchResultsController类用于管理表视图的数据，即将Core Data的持久化存储在表视图中，并对这些数据进行管理：增删改查
* 管理 undo/redo操作
* 检查托管对象的属性值是否正确

###*CoreData的6个成员对象*

1. NSManageObject：被管理的数据记录Managed Object Model是描述应用程序数据模型，这个模型包含实体(Entity),特性(Property)，读取请求(Fetch Request)等
2. NSManageObjectContext：管理对象的上下文，持久化存储模型对象，参与数据对象进行各种操作的全过程，并检测数据对象的变化，以提供对undo/redo的支持及更新绑定到数据的UI
3. NSPersistentStoreCoordinator：连接数据库的Persistent Store Coordinator,相当于数据文件管理器，处理底层的对数据文件的读取和写入，一般我们与这个没有交集
4. NSManagedObjectModel：被管理的数据模型，数据结构
5. NSFetchRequest：数据请求
6. NSEntityDescription：表格实体结构，还需知道. xcdatamodel文件编译后为`.momd`或者`.mom`文件。

### Core Data 的功能
* 对于KVC和KVO完整且自动化的支持，除了为属性整合KVO和KVC访问方法外，还整合了适当的集合访问方法来处理多值关系
* 自动验证属性值
* 支持跟踪修改和撤销操作
* 关系维护，CoreData管理数据的关系传播，包括维护对象之间的一致性
* 在内存上和界面上分组，过滤，组织数据
* 自动支持对象存储在外部数据仓库的功能
* 创建复杂请求：无需动手写SQL语句，在获取请求(Fetch request)中关联 NSPredicate,NSPredicated支持基本功能，想关子查询和其它高级的sql特性。它支持正确的Unicode编码，区域感知查询，排序和正则表达式
* 延迟操作：CoreData使用懒加载方式减少内存负载，还支持部分实体化延迟加载和复制队形的数据共享机制
* 合并策略：COreData内置版本跟踪和乐观锁来支持多用户写入冲突的解决，其中，乐观锁就是对局冲突进行检测，若冲突就返回冲突的信息
* 数据迁移：CoreData的Schema Migration工具尅简化对应数据库结构变化的任务，在某些情况下允许你执行高效率的数据库原地迁移工作
* 可选择针对程序Controller层的集成，来支持UI的显示同步core data在iphone os之上，提供NSFetchedResultsController对象来做相关工作，在Mac OS上我们用Cocoa提供的绑定机制来完成的

## 对象可以被Copy的条件
只有实现了NSCopying和NSMutableCopying协议的类的对象才能被拷贝分为不可变拷贝和可变拷贝，

* NSCopying协议方法为:

```
- (id)copyWithZone:(NSZone *)zone {
 MyObject *copy = [[[self class] allocWithZone: zone] init]; copy.username = [self.username copyWithZone:zone]; return copy;
}
```

## 在某个方法中 self.name = _name，name = _name 它 们有区别吗,为什么?

* 前者是存在内存管理的setter方法赋值，它会对_name对象进行保留或者拷贝操作
* 后者是普通赋值
* 一般来说，在对象的方法里成员变量和方法都是可以访问的，我们通常会重写setter方法来执行某些额外的工作，比如说，外部传一个模型过来，那么我会直接重写setter方法，当模型传来时，也就意味着数据发生了变化，那么视图也需要更新显示，则在赋值新模型的同时也去刷新UI

## 解释self=[super init]方法

* 容错处理，当父类初始化失败，会返回一个nil,表示初始化事变，由于继承的关系，子类是需要拥有父类的实例和行为，因此，我们必须先初始化父类，然后再初始化子类

## 定义属性时,什么时候用 assign、retain、copy 以及它们的之间的区别

* assign:普工赋值，一般常用于基本数据类型，常见委托设计模式，一次来防止循环引用（我们成为弱引用）
* retain:保留计数，获得到了对象的所有权，引用计数在原有的基础上加1
* copy:一般认为，是在内存中重新开辟了一个新的内存空间，用来存储新的对象，和原来的对象是两个不同的地址，引用计数分别1.当时当copy对象为不可变对象时，那么copy的作用相当于retain,因此，这样可以节约内存空间

### 堆和栈的区别

* 栈区(stack)由编译器自动分配释放，存放方法（函数）的参数值，局部变量的值等，栈是由低地址扩展的数据结构，是以一块连续的内存的区域。即栈顶的地址和栈的最大容量是系统预先规定好的
* 堆区（heap）：一般是由程序员分配释放，弱程序员不释放，程序结束时由OS回收，向高地址扩展的数据结构，是不连续的内存区域，从而堆获得的空间比较灵活
* 碎片问题：对于堆来讲，频繁的new/delete势必会造成内存空间的不连续，从而造成大量的碎片，使的程序效率降低。对于栈来讲，则不会存在这个问题，因为栈是先进后出的队列，他们是如此的一一对应，以至于永远都不能有一个内存块从栈中间弹出
* 分配方式：堆都是动态分配的，没有静态分配的堆。栈有2种分配方式：静态分配和动态分配。静态分配是编译器完成的，比如局部变量的分配。动态分配是由alloc函数进行分配，但是栈的动态分配和堆是不同的，它的动态分配是由编译器进行释放，无需我们手工实现
* 分配效率：栈是机器系统提供的数据结构，计算机会在底层对栈提供支持：分配专门的寄存器放栈的地址，压栈出栈都有专门的指令执行，这就决定了栈的效率比较高。堆是C/C++函数库提供的，它的机制是很复杂的
* 全局区(静态区)(static):全局变量和静态变量的存储是放在一块的。初始化的全局变量和静态变量在一块区域，未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。程序结束后由系统释放
* 文字常量区：常量字符串就是存放在这里的额，程序结束后由系统释放
* 程序代码区：存放函数体的二进制代码

## 怎样使用performSelector传入3个以上的参数，其中一个为结构体
* 因为系统提供的performSelector的API中，并没有提供三个参数，因此，我们只能传数组或者字典，但是数组或者字典只有存入对象类型，而结构体并不是对象类型，我们只能通过对象放入结构作为属性来传过去了。

```
 - (id)performSelector:(SEL)aSelector; 
 - (id)performSelector:(SEL)aSelector withObject:(id)object; 
 - (id)performSelector:(SEL)aSelector withObject:
    (id)object1 withObject:(id)object2;
```

具体实现如下:

```
typedef struct HYBStruct {
int a;
int b;
} *my_struct;
@interface HYBObject : NSObject
@property (nonatomic, assign) my_struct arg3;
@property (nonatomic, copy)  NSString *arg1;
@property (nonatomic, copy) NSString *arg2;

@end
@implementation HYBObject

// 在堆上分配的内存，我们要手动释放掉- (void)dealloc {
free(self.arg3);

}@end
```

测试:

```
my_struct str = (my_struct)(malloc(sizeof(my_struct)));
str->a = 1;
str->b = 2;
HYBObject *obj = [[HYBObject alloc] init];
obj.arg1 = @"arg1";
obj.arg2 = @"arg2";
obj.arg3 = str; 
[self performSelector:@selector(call:) withObject:obj]; 
// 在回调时得到正确的数据的- (void)call:(HYBObject *)obj { NSLog(@"%d %d", obj.arg3->a, obj.arg3->b);
}
```

## UITableViewCell有个UIlabel,显示NSTimer实现的秒表时间，手指滚动cell过程中，label是否刷新，为什么？

这是否刷新取决于timer加入到run loop中的mode是什么。Mode主要是用来指定事件在运行循环中的优先级的，分为：

* NSDefaultRunLoopMode（kCFRunLoopDefaultMode）：默认，空闲状态
* UITrackingRunLoopMode：ScrollView 滑动时切换到该Mode
* UIInitializationRunLoopMode：run loop启动时，会切换到该Mode
* NSRunLoopCommonModes（kCFRunLoopCommonModes）:Mode集合苹果公开提供的Mode有两个：
* NSDefaultRunLoopMode（kCFRunLoopDefaultMode）
* NSRunLoopCommonModes（kCFRunLoopCommonModes）
* 在编程中，如果我们把一个NStimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环中的时候，ScrollView滚动的过程中会因为mode的切换，而导致NSTimer将不再被调度，当我们滚动的时候，也希望不调度，那就应该使用该模式。
但是，我们希望在滚动的时候，定时器也要回调，那就应该使用common mode

## 对于单元格重用的理解
* 当cell滑动屏幕时，系统会把这个单元格添加到重用队列中，等待被重用，当有新单元格从屏幕外划入屏幕内部时，从重用队列中找看有没有可重用的单元格，若有，就直接用，没有就重新创建一个

## 解决Cell重用的问题

* UITableview通过重用单元格来达到节省内存的额目的，通过为每个单元格制定一个重用标识（reuseidentifier），即指定了单元格的种类，以及当单元格滚出屏幕时，允许恢复单元格以便复用。对于不同种类的单元格使用不同的Id，对于简单的表格，一个标识符就够了
* 如一个TableView中又10个单元格，但屏幕最多显示4个，实际上iPHone只为其分配了4个单元格的内存，没有分配10个，当关东单元格时，屏幕内显示的单元格重复使用这4个内存。实际上分配的cell的格式为屏幕最大显示数，当有新的cell进入屏幕时，会随你调用已经滚出屏幕的cell所占的内存，这就是cell的重用
* 对于多变的自定义cell,这种重用机制会导致内容出错，为解决这种出错的方法，把原来的


```
UITableViewCell *cell = [tableview dequeueReusableCellWithIdentifier:defineString]
修改为：UITableViewCell *cell = [tableview cellForRowAtIndexPath:indexPath];
```

这样就解决掉cell重用机制导致的问题，但是数据量多的情况，会有性能问题


## 有a、b、c、d 4个异步请求，如何判断a、b、c、d都完成执行？如果需要a、b、c、d顺序执行，该如何实现？

对于这4个异步请求，要判断都执行完成最简单的方式就是通过GCD的group来实现：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{ /*任务a */ });
dispatch_group_async(group, queue, ^{ /*任务b */ });
dispatch_group_async(group, queue, ^{ /*任务c */ }); 
dispatch_group_async(group, queue, ^{ /*任务d */ }); 
dispatch_group_notify(group,dispatch_get_main_queue(), ^{ // 在a、b、c、d异步执行完成后，会回调这里});


```

* 当然，我们还可以使用非常老套的方法来处理，通过4个变量来标识a,b,c,d四个人物是否完成，然后在runloop中让其等待，当完成时才退出runloop.但是这样做会让后面的代码得不到执行，直到Runloop执行完成
* 解释：要求顺序执行，那么可以将任务放到串行队列中，自然就是按顺序来异步执行了。

## 使用block有什么好处？使用NSTimer写出一个使用block显示（在UIlabel上）秒表的代码
* 代码紧凑，传值，回调都很方便，省去了写代理的很多代码
* NSTimer封装成block
* 实现方法:

```
NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1.0
                              repeats:YES
                             callback:^() {
  weakSelf.secondsLabel.text = ...
}
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes]
```

## 线程和进程的区别和联系？
* 一个程序至少要有进程，一个进程至少要有一个线程
* 进程：资源分配的最小独立单元，进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动，进程是系统进行资源分配和调度的一个独立单位
* 线程：进程下的一个分支，是进程的实体，是CPU调度和分派的基本单元，它是比进程更小的能独立运行的基本单位，线程自己基本不拥有系统资源，只拥有一点在运行中不可少的资源（程序计数器，一组寄存器，栈）但是它可与同属一个进程的其他线程共享进程所拥有的全部资源
* 进程和线程的主要差别在于他们是不同的操作系统资源管理方式。进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其他进程产生影响，而线程只是一个进程中的不同的执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于这个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源比较大，效率要差一些
* 但对于一些要求同事进行并且又要共享某些变量的并发造作，只能用线程，不能用进程

## 多线程编程
* NSThread:当需要进行一些耗时操作时会把耗时的操作放到线程中，线程同步：多个线程同事访问一个数据会出问题，NSlock,线程同步块，@synchronized(self){}。
* NSOperationQueue操作队列（不需要考虑线程同步问题）。编程的重点都放在main里面，NSInvocationOperation, BSBlockOperation，自定义Operaton.创建一个操作绑定相应的方法，当把操作添加到操作队列中时，操作绑定的方法就会自动执行了，当把操作添加到队列中时，默认会调用main方法。
* GCD（`Grand Central Dispatch`）宏大的中央调度，串行队列，并发队列，主线程队列
* 同步和异步：同步指第一个任务不执行完，不会开始第二个，异步是不管第一个有没有执行完，都开始第二个
* 串行和并行：串行是多个任务按照一定的顺序执行，并行是多个任务同事执行
* 代码是在分线程执行，在主线程刷新UI

### 多线程编程是防止主线程堵塞，增加运行效率的最佳方法

* Apple提供了NSOperation这个类，提供了一个优秀的多线程编程方法
* 一个NSOperationQueue操作队列，相当于一个线程管理器，而非一个线程，因为你可以设置这个想成管理器可以并行运行的线程数量等
* 多线程是一个比较轻量级的方法来实现单个应用程序内多个代码执行路径
* iPhoneOS下的主线程的堆栈大小是1M。第二个线程开始就是512KB，并且该值不能通过编译器开关或线程API函数来更改，只有主线程有直接修改UI的能力

## 定时器与线程的区别：
