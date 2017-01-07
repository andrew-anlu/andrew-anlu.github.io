---
layout: post
title: "swift3-associatedtype用法"
date: 2017-01-06 16:18:04 +0800
comments: true
categories: swift3
---

swift中的协议采用的是"Associated Types"的方式来实现泛型功能的，通过`associatedtype `关键字来声明一个类型的占位符作为协议定义的一部分，swift的协议不支持下面的定义方式:

```
protocol GeneratorType {
    public mutating func next() -> Element?
}

```

而是应该使用这样的定义方式:

```
protocol GeneratorType {
    associatedtype Element
    public mutating func next() -> Self.Element?
}
```

在Swift中，class,struct,enums都可以使用参数化类型来表达泛型的，只有在协议中需要使用associatedtype关键字来表达参数化类型。为什么协议不采用这样的语法形式呢？我查看了很多讨论，原因大概总结为两点:

1. 采用语法的参数化方式的泛型其实定义了整个类型的家族，在概念上这对于一个可以具体实现的类型（clas,stuct,enums）是由意义的，比方说Array.但对于协议来说，协议表达的含义是single的。你智慧实现一次
`GeneratorType `，而不会实现一个GeneratorType协议。接着又实现另外一个`GeneratorType `协议。
2. 协议在swift中有两个目的，第一个目的是用来实现多继承(swift语言被设计成单继承的)，第二个目的是强制实现者必须准守自己所指定的泛型约束。关键字`associatedtype `是用来实现第二个目的的，在`GeneratorType `中由`associatedtype `指定的Element,是用来控制next()方法的返回类型的。而不是用来指定`GeneratorType `的类型的。



