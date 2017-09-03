---
layout: post
title: "闭包-swift"
date: 2016-03-11 16:48:06 +0800
comments: true
categories: swift
---
##前言
闭包是自包含的函数代码块，可以在代码中被传递和使用。swift中的闭包与oc中的代码块(block)比较相似

闭包可以捕获和存储其所在上下文中任意常量和变量的引用。这就是所谓的闭包并包括着着这些常量和变量，俗称闭包。
swift会为您管理在捕获过程中涉及到所有内存操作。
<!--more-->

闭包采用如下三种形式之一

1. 全局函数是一个有名字但不会捕获任何值的闭包
2. 嵌套函数是一个有名字并可以捕获其封闭函数域内值的闭包
3. 闭包表达式是一个利用轻量级语法所写的可以捕获其上下文中常量或变量值的匿名闭包

swift的闭包表达式拥有简介的风格，并鼓励在常见场景下进行语法优化，主要优化如下:

1. 利用上下文推断参数和返回值类型
2. 隐式返回单表达式闭包，即单表达式闭包可以省略 return 关键字
3. 参数名称缩写
4. 尾随(Trailing)闭包语法

##闭包表达式(Closure Expressions)
嵌套函数是一个在较复杂函数中方便进行命名和定义字包含代码模块的方式。当然，有时候撰写小巧的没有完整定义和命名的类函数结构也是很有用处的，尤其是在您处理一些函数并需要将另外一些函数作为该函数的参数时。

闭包表达式是一种利用简介语法构建内联闭包的方式。闭包表达式提供一些语法优化，使得撰写闭包变得简单明了。下面的闭包表达式的例子通过使用几次迭代展示了 `sort(_:)`方法定义和语法优化的方式。
每一次迭代都用更简洁的方式描述了相同的功能。

###sort方法(The Sort Method)
Swift标准库提供了名为`sort`的方法，会根据您提供的用于排序的闭包函数将已知类型数组中的值进行排序。一旦排序排序完成，`sort(_:)`方法会返回一个与原数组大小相同，包含同类型元素且元素已正确排序的新数组。原数组不会被`sort(_:)`方法修改。

下面的闭包表达式示例使用`sort(_:)`方法对一个`String`类型数组进行字母逆序排序，以下是初始数组值:

```
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
```

`sort(_:)`方法接受一个闭包，该闭包函数需要传入与数组元素类型相同的两个值，并返回一个布尔类型的值表明当排序结束后传入的第一个参数排在第二个参数前面还是后面。如果第一个参数值出现在第二个参数值前面，排序闭包函数需要返回 true,反之返回false。

该例子对一个 `String`类型的数组进行排序，因此排序闭包函数类型需为(String,String)->Bool.
提供排序闭包函数的一种方式是撰写一个符合其类型要求的普通函数，并将其作为`sort(_:)`方法的参数传入:

```
func backwards(s1: String, s2: String) -> Bool {
    return s1 > s2
}
var reversed = names.sort(backwards)
// reversed 为 ["Ewa", "Daniella", "Chris", "Barry", "Alex"]
```

如果第一个字符串(s1)大于第二个字符串(s2)，`backwards(_:_:)`函数返回`true`,表示在新的数组中`s1`应该出现在`s2`前。

###闭包表达式语法(Closure Expression Syntax)
闭包表达式语法有如下一般形式:

```
{(parameters) -> return type in
	statements
}
```
闭包表达式语法可以使用常量，变量和`inout`类型作为参数，不能提供默认值。也可以在参数列表的最后使用可变参数。
元组也可以作为参数和返回值.

下面的例子展示了之前 `backwards(_:_:)`函数对应的闭包表达式版本的代码:

```
reversed = names.sort({ (s1:String , s2:String) -> Bool in 
	return s1 > s2
})
```
需要注意的是内联闭包参数和返回值类型声明与`backwards(_:_:)`函数类型声明相同。在这两种方式中，都写成了(s1:String,s2:String) - >Bool.然而在内联闭包表达式中，函数和返回值类型都卸载大括号内，而不是大括号外。

闭包的函数体部分由关键字  in 引入。该关键字表示闭包的参数和返回值类型定义已经完成，闭包函数体即将开始。

由于这个闭包函数体部分如此短，以至于可以将其改写成一行代码:

```
reversed = names.sort({(s1:String,s2:String)->Bool in 
	return s1>s2
	})
```

该例中`sort(_:)`方法的整体调用保持不变，一对圆括号仍然包裹住了方法的整个参数。然后，参数现在变成了内联闭包。

###根据上下文推断类型(Inferring Type from Context)
因为排序闭包函数是作为`sort(_:)`方法的参数传入的，swift可以推断其参数和返回值类型。`sort(_:)`方法被一个字符串数组调用，因此其参数必须是(String,String)->Bool类型的函数。这意味着(String,String)和Bool类型并不需要作为闭包表达式定义的一部分。因为所有的类型都可以被正确推断，返回箭头(->)和围绕在参数周围的括号也可以被省略:

```
reversed = names.sort({s1,s2 in return s1 > s2})
```

实际上任何情况下，通过内联闭包表达式构造的闭包作为参数传递给函数活方法时，都可以推断出闭包的参数和返回值类型。这意味着闭包作为函数或者方法的参数时，您几乎不需要利用完整格式构造内联闭包。

尽管如此，您仍然可以明确写出有着完整格式的闭包。如果完整格式的闭包能提高代码的可读性，则可以采用完整格式的闭包。而在`sort(_:)`方法这个例子里，闭包的目的就是排序。由于这个闭包是为了处理字符串数组的排序，因此读者能够推测出这个闭包是用于字符串处理的。

###单表达式闭包隐式返回
单行表达式闭包可以通过省略`return`关键字来隐式返回单行表达式的结果，如上版本的例子可以改写成:

	reversed=names.sort({s1,s2 in s1 > s2})
在这个例子中，`sort(_:)`方法的第二个参数函数类型明确了闭包必须返回一个`Bool`类型之。因为闭包函数体只包含了一个单一表达式(`s1>s2`),改表达式返回`Bool`类型值，因此这里没有歧义，`return`关键字可以省略

###函数名称缩写
swift自动为内联闭包提供了参数名称缩写功能，您可以额直接通过`$0,$1,$2`来顺序调用闭包参数，以此类推。
如果您在闭包表达式中使用参数名称缩写，您可以在闭包参数列表中省略对其的定义，并且对应参数名称缩写的类型会通过函数类型进行判断。`in`关键字同样可以被省略额，因为此时闭包表达式完全有闭包函数体构成：

	reveserd = names.sort({$0 > $1})
	
在这个例子中，$0和$1表示闭包中第一个和第二个`String`类型的参数。
###运算符函数(OPerator Functions)
实际上还有一种更简短的方式来撰写上面例子中的闭包表达式。Swift的`String`类型定了关于大于号(>)的字符串实现，其作为一个函数接受两个`String`类型的参数并返回`Bool`类型的值。而这正好与`sort(_:)`方法的第二个参数需要的函数类型相符合。因此，您可以简单传递一个大于号，Swift可以自动推断出您想使用大于号的字符串函数实现：
	
	reversed = names.sort(>)

##尾随闭包(Trailing Closures)
如果您需要将一个很长的闭包表达式作为最后一个参数传递给函数，可以使用尾随闭包来增强函数的可读性。尾随闭包是一个书写在函数括号之后的彼表表达式，函数支持将其作为最后一个参数调用:

```
func someFunctionThatTakesAClosure(closure: () -> Void){
	   //函数体部分
	}
  // 以下是不使用尾随闭包进行函数调用
  someFunctionThatTakesAClosure({
    // 闭包主体部分
  })

  // 以下是使用尾随闭包进行函数调用
 someFunctionThatTakesAClosure() {
     // 闭包主体部分
 }
```

上面的sort排序可以改为:
	
	reversed = names.sort(){$0 > $1}
如果函数只需要闭包表达式一个参数，当您使用尾随闭包时，您甚至可以吧()省略掉
	
	reversed = names.sort { $0 > $1}
	
当闭包非常长以至于不能再一行中进行书写时，尾随闭包变得非常有用。举例来说，swift的`array`类型有一个`map(_:)`方法，其获取一个闭包表达式作为其唯一参数。该闭包函数会为数组中的每一个元素调用，并返回该元素所映射的值。具体的映射方式和返回值类型由闭包来指定。

当提供给数组的闭包应用于每个数组元素后，`map(_:)`方法将返回一个新的数组，数组中包含了原数组中的元素-对应的映射后的值

下例介绍了如何在`map(_:)`方法中使用尾随闭包将`Int`类型数组`[16,58,510]`转换为包含对应`String`类型的值的数组`["OneSix", "FiveEight", "FiveOneZero"]`

```
let digitNames = [
    0: "Zero", 1: "One", 2: "Two",   3: "Three", 4: "Four",
    5: "Five", 6: "Six", 7: "Seven", 8: "Eight", 9: "Nine"
]
let numbers = [16, 58, 510]
```

如上代码创建了一个数字位和它们英文版名字想映射的字典。同时还定义了一个准备转换为字符串数组的整形数组。
您现在可以通过传递一个尾随闭包给`numbers`的`map(_:)`方法来创建对应的字符串版本数组:

```
let strings = numbers.map {
    (var number) -> String in
    var output = ""
    while number > 0 {
        output = digitNames[number % 10]! + output
        number /= 10
    }
    return output
}
// strings 常量被推断为字符串类型数组，即 [String]
// 其值为 ["OneSix", "FiveEight", "FiveOneZero"]
```
`map(_:)`为数组中每一个元素调用了闭包表达式。您不需要指定闭包的输入参数`number`的类型，因为可以通过要映射的数组类型进行判断。

在该例子中，闭包`number`参数被声明为一个变量参数，因此可以在闭包函数体内对齐进行修改，而不用再定义一个新的局部变量并将`number`的值赋值给它。闭包表达式指定了返回类型为String,以表明存储映射值的新数组类型为`String`.

闭包表达式在每次被调用的时候创建了一个叫做`output`的字符串并返回。
###捕获值(Capturing Values)
闭包可以在其被定义的上下文中捕获常量或变量。及时定义这些常量和变量的原作用域已经不存在，闭包仍然可以在闭包函数体内引用和修改这些值。

Swift中，可以捕获值的彼表的最简单形式是嵌套函数，也就是定义在其他函数的函数体内的函数。嵌套函数可以捕获其外部函数所有的参数一级定义的常量和变量。

举个例子，这有一个叫做`makeIncrementor`的函数，其包含了一个叫做`incrementor`的嵌套函数。嵌套函数`incrementor()`从上下文中捕获了两个值，`reunningTotal`和`amount`。捕获这些值之后，`makeIncrementor`将`incrementor`作为闭包返回。每次调用`incrementor`时，其会以`amout`作为增量增加`runningTotal`的值

```
func makeIncrementor(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementor() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementor
}
```
`makeIncrementor`返回类型为`()->Int`.这意味着其返回一个函数，而不是一个简单类型的值。该函数在每次调用时不接受参数，值返回一个`Int`类型的值。
`makeIncrementer(forIncrement:)`函数定义了一个初始值为0的整形变量`runningTotal`，用来存储当前跑步总数。该值通过`incrementor`返回。

`makeIncrementor(forIncrement:)`有一个`Int`类型的参数，其外部参数名为`forIncrement`,内部参数名为`amount`,该参数表示每次`incrementor`被调用时`runingTotal`将要增加的量。

嵌套函数`increment`用来执行实际的增加操作。该函数简单滴使用`runningTotal`增加`amount`，并将其返回。
如果我们单独看这个函数，会发现看上去不同寻常:

```
func incrementor() -> Int {
    runningTotal += amount
    return runningTotal
}

```
`incrementer()`函数并没有任何参数，但是在函数体访问了`runningTotal`和`amount`变量。这是因为它从外围函数捕获了`runninTotal`和`amount`变量的引用。捕获引用保证了`runningTotal`和`amount`变量在调用完`makeIncrementer`后不会消失，并且保证了下一次执行`incrementer`函数时，`runningTotal`依旧存在。



下面是一个使用`makeIncrementor`的例子:

	let incrementByTen = makeIncrementor(forIncrement: 10)


该例子定义了一个叫做 `incrementByTen`的常量，该常量指向一个每次调用会将`runningTotal`变量增加10的 `incrementor`函数。调用这个函数多次可以得到如下结果:

```
incrementByTen()
// 返回的值为10
incrementByTen()
// 返回的值为20
incrementByTen()
// 返回的值为30
```
如果您蒋欢了另一个`incrementor`，它会有属于它自己的一个全新的，独立的`runnintTotal`变量的引用:

```
let incrementBySeven = makeIncrementor(forIncrement: 7)
incrementBySeven()
// 返回的值为7
```
再次调用原来的`incrementByTen`会在原来的变量`runningTotal`上继续增加值,该变量和`incrementBySeven`中捕获的变量没有任何联系:

	incrementByTen()
    // 返回的值为40  
    
##闭包是引用类型

上面的例子中，`incrementBySeven`和`incrementByTen`是常量，但是这些常量指向的闭包仍然可以增加其捕获的变量的值。这是因为函数和闭包都是引用类型。

无论您将函数或闭包赋值一个常量还是变量，您实际上都是将常量或者变量的值设置为对应函数或闭包的引用。上面的例子中，指向闭包的引用`incrementByTen`是一个常量，而并非闭包内容本身。

这也意味着如果您将闭包赋值给两个不同的常量或变量，两个值都会指向同一个闭包:

```
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen()
// 返回的值为50
```

##非逃逸闭包(Nonescaping Closures)
当一个闭包作为参数传到一个函数中，但是这个闭包在函数返回之后才被执行，我们称该闭包从函数中逃逸。当你定义接受闭包作为参数的函数时，你可以在参数名之前标注`@noescape`,用来指明这个闭包是不允许"逃逸"出这个函数的。将闭包标注`@noescape`能使编译器知道这个闭包的生命周期,从而可以进行一些比较激进的优化。

```
func someFunctionWithNoescapeClosure(@noescape closure: () -> Void) {
    closure()
}
```
举个例子,`sort(_:)`方法接受一个用来进行元素比较的闭包作为参数。这个参数被标注了`@noescape`，因为它确保自己在排序结束之后就没用了。

一种能使闭包'逃逸'
出函数的方法是，将这个闭包保存在一个函数外部定义的变量中。举个例子，很多异步操作的函数接受一个闭包参数作为 completion handler.这类函数会在异步操作开始之后立刻返回，但是闭包直到异步操作结束后才会被调用。在这种情况下，闭包需要“逃逸”出函数，因为闭包需要在函数返回之后被调用。例如:

```
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: () -> Void) {
    completionHandlers.append(completionHandler)
}
```
`someFunctionWithEscapingClosure(_:)`函数接受一个闭包作为参数，该闭包被添加到一个函数外定义的数组中。如果你视图将这个参数标注为`@noescape`，你将会得到一个编译错误。

将闭包标注为`@noescape`使你能在闭包中隐式地引用`self`.


```
class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNoescapeClosure { x = 200 }
    }
}

let instance = SomeClass()
instance.doSomething()
print(instance.x)
// prints "200"

completionHandlers.first?()
print(instance.x)
// prints "100
```



##自动闭包(Autoclosures)
自动闭包是一种自动创建的闭包，用于包装传递给函数作为参数的表达式。这种闭包不接受任何参数，当它被调用的时候，会返回被包装在其中的表达式的值。这种便利语法让你能够用一个普通的表达式来代替显式的闭包，从而省略闭包的花括号。

自动闭包让你能够延时求值，因为代码段不会被执行直到你显式的调用这个闭包。延迟求值对于那些有副作用和代价昂贵的代码来说是很有益处的，因为你能控制代码什么时候执行。下面的代码展示了闭包如何延时求值。

```
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// prints "5"

let customerProvider = { customersInLine.removeAtIndex(0) }
print(customersInLine.count)
// prints "5"

print("Now serving \(customerProvider())!")
// prints "Now serving Chris!"
print(customersInLine.count)
// prints "4"
```
尽管在闭包的代码中，``customersInLine ``的第一个元素被移除了，不过在闭包调用之前，这个元素不会被移除的。如果这个闭包永远不被调用，那么闭包里卖弄的表达式将永远不会执行，那意味着列表中的元素永远不会被删除。请注意，`customeerProvider`的类型不是`String`,而是`() -> String`,一个没有参数且返回值为`String`的函数。
将闭包作为参数传递给函数时，你能获的同样的延时求值行为.



```
// customersInLine is ["Alex", "Ewa", "Barry", "Daniella"]
func serveCustomer(customerProvider: () -> String) {
    print("Now serving \(customerProvider())!")
}
serveCustomer( { customersInLine.removeAtIndex(0) } )
// prints "Now serving Alex!"
```
`serveCustomer(_:)`接受一个返回顾客名字的显式的闭包。下面的这个版本的 serveCustomer(_:)完成了相同的操作，不过它并没有接受一个显示的闭包，而是通过该将参数标记为`@autoclosure`来接收一个自动闭包。现在你可以将该函数当做接受`String`类型参数的函数来调用。`customerProvider`参数将自动转化为一个闭包，因为该参数标记了`@autoclosure`特性。

```
// customersInLine is ["Ewa", "Barry", "Daniella"]
func serveCustomer(@autoclosure customerProvider: () -> String) {
    print("Now serving \(customerProvider())!")
}
serveCustomer(customersInLine.removeAtIndex(0))
// prints "Now serving Ewa!"
```

`@autoclosure`特性暗含了`@noescape`特性。如果你想让这个闭包可以"逃逸"，则应该使用`@autoclosure(escaping)`特性。

```
// customersInLine is ["Barry", "Daniella"]
var customerProviders: [() -> String] = []
func collectCustomerProviders(@autoclosure(escaping) customerProvider: () -> String) {
    customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.removeAtIndex(0))
collectCustomerProviders(customersInLine.removeAtIndex(0))

print("Collected \(customerProviders.count) closures.")
// prints "Collected 2 closures."
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// prints "Now serving Barry!"
// prints "Now serving Daniella!"
```

在上面的代码中，`collectCustomerProviders(_:)`函数并没有调用传入的`cusomerProvider`闭包，而是将闭包追加到了`customerProviders`数组中。这个数组定义在函数作用域范围外，这意味着数组内的闭包将会在函数返回之后被调用，因此，`customerProvider`参数必须允许“逃逸”出函数作用域.





 