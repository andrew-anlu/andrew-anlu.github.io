---
layout: post
title: "swift-泛型"
date: 2016-03-24 10:11:56 +0800
comments: true
categories: swift
---

泛型代码可以让你编写使用自定义需求以及任意类型的灵活可冲中的函数和类型，它可以让你避免重复的代码，用一种清晰和抽象的方式来表达代码的意图。
<!--more-->

泛型是swift的强大特性之一，许多swift标准库是通过泛型代码构建的。事实上，泛型的使用贯穿饿了正本语言手册，只是你可能没有发现而已。例如,swift的`Array`和`Dictionary`都是泛型集合。你可以创建一个`Int`数组，也可以创建一个`String`数组，甚至可以是任意其它Swift类型的数组。同样的，你也可以创建存储任意指定类型的字典。



##泛型所解决的问题
下面是一个标准的非泛型函数 `swapTwoInts(_:_:)`，用来交换两个`Int`值:

```
func swapTwoInts(inout a: Int, inout _ b: Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

这个函数使用输入输出函数(`inout`)来交换`a`和`b`的值。

`swapTwoInts(_:_:)`函数交换`b`的原始值到`a`,并交换`a`的原始值到`b`,你可以调用这个函数交换两个`Int`变量的值:

```
var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
// 打印 “someInt is now 107, and anotherInt is now 3”
```
诚然，`swapTwoInts(_:_:)`函数挺有用，但是它只能交换`Int`值，如果你想要交换两个`String`值或者`Double`值，就不能不写更多的函数，例如`swapTwoStrings(_:_:)`和`swapTwoDoubles(_:_:)`,如下所示:

```
func swapTwoStrings(inout a: String, inout _ b: String) {
    let temporaryA = a
    a = b
    b = temporaryA
}

func swapTwoDoubles(inout a: Double, inout _ b: Double) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

你可能注意到`swapTwoInts(_:_:) 和 swapTwoDoubles(_:_:)`,的函数功能都是相同的，唯一不同之处就在于传入的变量类型不同，分别是 int,String,Double.

在实际的应用中，通常需要一个更实用更灵活的函数来交换两个任意类型的值，幸运的是，泛型代码帮你解决了这种问题。

>*注意*
>在上面的三个函数中，`a`和`b`类型相同，如果`a`和`b`类型不同，那他们俩就不能交换值。Swift是类型安全的语言，所以不会允许一个`String`类型的变量和一个`Double`类型的变量互换值。视图这样做将导致编译错误。


##泛型函数
泛型函数可以适用于任何类型，下面的`swapTwoValues(_:_:)`函数是上面三个函数的泛型版本:

```
func swapTwoValues<T>(inout a:T,inout _ b:T){
 let temporaryA=a;
 a=b;
 b= temporaryA;
}
```

`swapTwoValues(_:_:)`的函数主体和`swapTwoInts(_:_:)`函数是一样的。他们只在第一行有所不同，如下所示:

```
func swapTwoInts(inout a: Int, inout _ b: Int)
func swapTwoValues<T>(inout a: T, inout _ b: T)
```

这个函数的泛型版本使用了占位类型名(在这里用字母`T`来表示)来代替实际的类型名(例如`Int`,`String`或者`Double`).占位类型名没有指明`T`必须是什么类型，但是它指明了`a`和`b`必须是同一类型`T`,而不论`T`代表什么类型，只有`swapTwoValues(_:_:)`函数在调用时，才能根据所传入的实际类型决定`T`所代表的类型。

另外一个不同之处在于这个泛型函数名后面跟着占位类型名(T),而且使用尖括号括起来的(`<T>`).这个尖括号告诉Swift那个`T`是`swapTwoValues(_:_:)`函数定义的一个占位类型名，因此swift不会去查找名为`T`的实际类型。


swapTwoValues(_:_:) 函数现在可以像 swapTwoInts(_:_:) 那样调用，可以传入任意类型的值，只要两个值的类型相同。`swapTwoValues(_:_:)`函数调用时，`T`所代表的类型都会由传入的值的类型判断出来.

在下面的两个例子中，`T`分别代表`Int`和`String`

```
var someInt = 3
var anotherInt = 107
swapTwoValues(&someInt, &anotherInt)
// someInt is now 107, and anotherInt is now 3

var someString = "hello"
var anotherString = "world"
swapTwoValues(&someString, &anotherString)
// someString is now "world", and anotherString is now "hello"
```

##类型参数
在上面的`swapTwoValues(_:_:)`例子中，占位类型`T`是类型参数的一个例子。类型参数指定并命名一个占位类型，并且紧随在函数名后面，使用一对尖括号括起来(<T>).

一旦一个类型参数被指定，你可以用它来定义一个函数的参数类型，或者作为函数的返回类型，还可以用作函数主题中的注释类型。在这些情况下，类型参数在函数调用时被实际类型所代替。


##命名类型参数
在大多数情况下，类型参数具有一个描述性名字，例如`Dictionary<Key,Value>`中的key和value,以及`Array<Element>`中的`Element`，这可以告诉阅读代码的人这些类型参数和泛型函数之间的关系。然而，当他们之间的关系没有意义时，通常使用单一的字母来命名，例如`T`,`U`,`V`，正如上面演示的`swapTwoValues(_:_:)`函数中的`T`一样.

##泛型类型
除了泛型函数，Swift还允许你定义泛型类型。这些自定义类，结构体和枚举可以适用于任何类型，如同`Array`和`Dictionary`的用法。

这部分内容将向你展示如何编写一个名为`stack`（栈）的泛型集合类型。栈是一系列值的有序集合，如`Array`类型，但它比Swift的array类型有更多的操作限制。数组允许对其中任意位置的元素执行插入或删除操作。而栈，只允许在集合的末端添加新的元素，称为入栈。同样滴，栈也只能从末端移除元素。


下面展示了一个如何编写一个非泛型版本的栈，在这种情况下是`Int`型的栈.

```
struct IntStack {
    var items=[Int]()
    
    mutating func push(item:Int){
        items.append(item);
    }
    
    mutating func pop()->Int{
        return items.removeLast();
    }
}
```

这个结构体在栈中使用一个名为`items`的`array`属性来存储值。`Stack`提供了两个方法:`push(_:)`和`pop()`，用来向栈中压入值以及从栈中移除值。这些方法被标记为`mutating`，因为他们需要修改结构体的`items`数组。

上面的`IntStack`结构体只能用于Int类型，不过可以定义一个泛型的`Stack`结构体，从而能够处理任意类型的值。

下面是相同代码的泛型版本:

```
struct Stack<Element> {
    var items=[Element]();
    
    mutating func push(item:Element){
        items.append(item);
    }
    
    mutating func pop()->Element{
        return items.removeLast();
    }
    
}
```

注意，`Stack`基本上和`IntStack`相同，只是占位类型参数`Element`代替了实际的`Int`类型。这种类型参数包裹在一对尖括号里(`<Element>`)，紧跟在结构体名后面、

`Element`为尚未提供的类型定义了一个占位名。这种尚未提供的类型可以在结构体定义中通过`Element`来引用。在这种情况下，`Element`在如下三个地方被用作占位符:

* 创建 items 属性，使用`Element`类型的空数组进行初始化
* 执行`psh(_:)`方法的单一参数`item`的类型必须是`Element`类型
* 指定`pop()`方法的返回值类型必须是`Element`类型.

由于`stack`是泛型类型，因此可以用来创建Swift中任意有效类型的栈，如同`Array`和`Dictionary`.

你可以通过在尖括号中写出栈中需要存储的数据类型来创建并初始化一个`Stack`实例。例如，要创建一个`String`类型的栈，可以写成`Stack<String>`：

```
var stackOfStrings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("dos")
stackOfStrings.push("tres")
stackOfStrings.push("cuatro")
// 栈中现在有 4 个字符串
```
移除并返回栈顶部的值


```
let fromTheTop = stackOfStrings.pop()
// fromTheTop 的值为 "cuatro"，现在栈中还有 3 个字符串
```


##扩展一个泛型类型
当你扩展一个泛型类型的时候，你并不需要在扩展的定义中提供类型的参数列表。更加方便的是，原始类型定义中声明的类型参数列表在扩展中可以直接使用，并且这些来自原始类型中的参数名称会被用作原始定义中类型参数的引用。

下面的例子扩展了泛型类型`Stack`，为其添加了一个名为`topItem`的只读计算型属性，它将会返回当前栈顶端的元素而不会将其从栈中移除:

```
extension Stack{
    var topItem:Element?{
        return items.isEmpty ? nil:items[items.count];
    }
}
```

`topItem`属性会返回一个`Element`类型的可选值。当栈为空的时候，`topItem`会返回`nil`;当栈不为空的时候，`topItem`会返回`nil`;当栈不为空的时候，`topItem`会返回`items`数组中的最后一个元素.

注意，这个扩展并没有定义一个类型参数列表。相反地，`Stack`类型已有的类型参数名`Element`，被用在扩展中表示计算型属性`topItem`的可选类型，`topItem`现在可以用来访问任意`Stack`实例的顶端元素而不是移除它

```
if let topItem = stackOfStrings.topItem {
    print("The top item on the stack is \(topItem).")
}
// 打印 “The top item on the stack is tres.”
```	

##类型约束
`swapTwoValues(_:_:)`函数和`Stack`类型可以作用于任何类型。不过，有的时候如果能将使用在泛型函数和泛型类型中的类型，强制约束为某种特定类型，将会是非常有用的。类型约束可以指定一个类型参数必须集成自特定类，或者符合一个特定的协议或者协议组合。

例如,Swift的`Dictionary`类型对字典的键的类型做了限制，在字典的描述中，字典的键必须是可哈希的。也就是说，必须有一种方法能作为其唯一的表示。`Dictionary`之所以需要其键是可哈希的，是为了便于检查字典是否已经包含了某个特定键的值。如无此要求，Dictionary将无法判断是否可以插入或者替换某个指定键的值，也不能查找到已经存储在字典中指定键的值。

这个要求强制加上了一个类型约束作用域`Dictionary`的键类型上，其键类型必须符合`Hashable`协议，这是swift标准库中定义的一个特定协议。所有的swfit基本类型(String,Int,Double)默认都是可哈希的。

当你创建自定义反省类型时，你可以定义你自己的类型约束，这些约束将提供更为强大的泛型编程能力。抽象概念，例如可哈希的。

##类型约束语法
你可以在一个类型参数名后面放置一个类名或者协议名，通过冒号分割，从而定义类型约束，它们将作为类型参数列表的一部分。这种基本的类型约束作用于泛型函数时的语法如下：

```
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // 这里是泛型函数的函数体部分
}
```

上面的这个函数有两个类型参数。第一个类型是参数`T`,有一个要求`T`必须是`SomeClass`自雷的类型约束；第二个类型参数`U`,有一个要求`U`必须符合`someProtocol`协议的类型约束.

##类型约束实践
这里有个名为`findStringIndex`的非泛型函数，该函数的功能在`String`值的数组中查找指定`String`值的索引。弱查找到匹配的字符串，`findStringIndex(_:_:)`函数返回该字符串在数组中的索引值，反之则返回`nil`:

```
func findStringIndex(array: [String], _ valueToFind: String) -> Int? {
    for (index, value) in array.enumerate() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

该函数可以用于查找字符串数组中的某个字符串:

```
let strings = ["cat", "dog", "llama", "parakeet", "terrapin"]
if let foundIndex = findStringIndex(strings, "llama") {
    print("The index of llama is \(foundIndex)")
}
// 打印 “The index of llama is 2”
```

如果只能查找字符串在数组中的索引，用处不是很大，不过，你可以写出相同功能的泛型函数`findIndex(_:_:)`,用占位类型`T`代替`String`类型。

下面展示了`findStringIndex(_:_:)`函数的泛型版本`findIndex(_:_:)`.请注意这个函数仍然返回`Int?`，那是因为函数返回的是一个可选的索引数，而不是从数组中得到一个可选值。需要提醒的是，这个函数无法通过编译，原因例子后面说明:

```

func findIndex<T>(array:[T], valueToField:T)->Int?{
    for (index, value)in array.enumerate(){
        if value == valueToField{
            return index;
        }
    }
    
    return nil;
}

```
上面所写的函数无法通过编译。这个问题出在相等性检查上，即`if vlaue == valueToField`。不是所有的Swift类型都可以用等号(==)进行比较。例如，如果你创建一个你自己的类活结构体表示一个复杂的数据模型，那么Swift无法猜到对于这个类或者结构体而言 “相等”意味着什么。正因为如此，这部分代码无法保证适用于每个可能的类型`T`,当你试图编译这部分代码的时候会出现相应的错误。

不过所有的这些并不会让我们无从下手。Swift标准库中定义了一个`Equatable`协议，该协议要求任何符合该协议的类型必须实现等式符号(==),从而对符合该协议的类型的任意两个值进行比较。所有的Swift标准类型自动支持Equatable协议。

任何`Equatable`类型都可以安全地使用在`findIndex(_:_:)`函数中，因为其保证支持等式操作符。为了说明事实，当你定义一个函数时，你可以定义一个`Equatable`类型约束作为类型参数定义的一部分:


```
func findIndex<T: Equatable>(array: [T], _ valueToFind: T) -> Int? {
    for (index, value) in array.enumerate() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

`findIndex(_:_:)`中的这个单一类型参数协作`T:Equatable`，也就意味着`任何符合Equatable`协议的`T`类型。

findIndex函数现在可以成功编译了。并且可以作用于任何符合`Equatable`的类型，如`Double`或`String`:

```
let doubleIndex = findIndex([3.14159, 0.1, 0.25], 9.3)
// doubleIndex 类型为 Int?，其值为 nil，因为 9.3 不在数组中
let stringIndex = findIndex(["Mike", "Malcolm", "Andrea"], "Andrea")
// stringIndex 类型为 Int?，其值为 2
```


##关联类型
下面例子定义了一个`Container`协议，该协议定义了关联类型`ItemType`:

```
protocol Container{
   typealias ItemType
    mutating func append(item:ItemType)
    
    var count:Int{get}
    
    subscript(i:Int)->ItemType{get}
}

```
`Container`协议定义了三个任何采纳协议的类型必须提供的功能：

* 必须可以通过`append`方法添加一个新元素到容器里。
* 必须可以通过`count`属性获取容易中元素的数量，并返回一个Int
* 必须可以通过接受`Int`索引值的下标检索到每一个元素

这个协议没有指定容易中元素该如何存储，以及元素必须是何种类型。这个协议只指定了三个任何采纳`Container`协议的类型必须是提供的功能。采纳协议的类型在满足这三个条件的情况下也可以提供其他额外的功能。


任何采纳`Container`协议的类型必须能够指定其存储的元素的类型，必须保证只有正确类型的元素可以加进容器中，必须明确通过其下标返回的元素类型。

为了定义这三个条件，`Container`协议需要在不知道容易中元素具体类型的情况下引用这种类型。

`Container`协议声明了一个关联类型`ItemType`，协作`typealias Itemtype`.这个协议无法定义`ItemType`是什么类型的别名，这个信息将留给采纳协议的类型来提供。尽管如此，`ItemType`别名提供了一种方式来引用`Container`中元素的类型，并将之用于`append`方法和下标，从而保证任何`Container`的预期行为都能够被执行。

下面采用复合`Container`协议:

```
struct IntStackContainer:Container {
    var items:[Int]
    
    mutating func push(item:Int){
     items.append(item)
    }
    
    mutating func pop()->Int{
        return items.removeLast();
    }
    
    //Container协议的实现部分
    typealias ItemType=Int
    
    mutating func append(item: ItemType) {
        self.push(item)
    }
    
    var count:Int{
      return items.count
    }
    
    subscript(i:Int)->Int{
        return items[i];
    }
    
}
```

`IntStackContainer`结构体实现了`Container`协议的三个要求，其原有功能也不会和这些要求冲突。

此外，`IntStackContainer`指定 ItemType为Int类型，即`typealias ItemType = Int`，从而将`Container`协议中抽象的`ItemType`类型转为具体的`Int`类型。

##通过扩展一个存在的类型来指定关联类型

Swift的`Array`已经提供了`append(_:_:)`方法，一个`count`属性，以及一个接受`Int`型索引值的可用来检索数组元素的下标。这三个功能都符合`Container`协议的要求，也就意味着你可以扩展`Array`去符合`Container`协议，只需简单滴声明`Array`采纳该协议即可。你可以通过一个空扩展来实现这点

```
extension Array:Container{}
```
如何上面的泛型`Stack`结构一样，array的`append(_:)`方法和下标确保了Swift可以推断出`ItemType`的类型，定义了这个扩展后，你可以将任意`Array`当做`Container`来使用
