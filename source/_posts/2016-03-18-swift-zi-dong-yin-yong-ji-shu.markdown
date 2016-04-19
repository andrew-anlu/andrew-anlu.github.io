---
layout: post
title: "swift-自动引用计数"
date: 2016-03-18 11:17:07 +0800
comments: true
categories: swift
---

swift使用自动引用计数(ARC)机制来跟踪和管理你的应用程序的内存。通常情况，Swift内存管理机制会一直起作用，你无需自己来考虑内存的管理。ARC会在类的实例不再需要时，自动释放其占用的内存。
然后在少数的情况下，为了能帮助你管理内存，ARC需要更多的代码之间关系的信息。
<!--more-->

> **注意:**
> 引用计数仅仅应用于类的实例。结构体和枚举类型是属于值类型，不是引用类型，也不是通过引用的方式存储和传递.
>

## 自动引用计数的工作机制
当你每次创建一个类的新的实例的时候，arc会分配一块内存来存储该实例信息。内存中会包含实例的类型信息，以及这个实例所有相关的属性的值。
此外，当实例不在被使用时，ARC释放实例所占用的内存。这确保了不再被使用的实例，不会一直占用内存空间。

然而，当ARC回收和释放了正在被使用中的实例，该实例的属性和方法将不再被访问和调用。实际上，如果你还试图访问这个实例，你的应用程序很可能会崩溃。

为了确保使用中的实例不会被销毁，ARC会跟踪和计算每一个实例正在被多少属性，常量和变量所引用。哪怕实例的引用数为1，ARC都不会销毁这个实例。

为了使上述成为可能，无论你将实例赋值给属性，常量或者变量，他们都会
创建此实例的`强`引用,之所以称为强引用，是因为它会将实例牢牢地包吃住，只要强引用还在，实例是不允许被销毁的。



## 自动引用计数实践

下面的例子展示了自动引用计数的工作机制。例子以一个简单的`Person`类开始，并定义了一个叫`name`的常量属性:

```
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    deinit {
        print("\(name) is being deinitialized")
    }
}
```
Person类有一个构造函数，此构造函数为实例的`name`属性赋值，并打印一个消息表明初始化过程生效。`Person`类也拥有一个析构函数，这个析构函数在实例被销毁时打印一条消息。

接下来的代码片段定义了三个类型为`Person?`的变量，为新的`Person`实例建立了多个引用。由于这些变量是被定义为可选类型，它们的值会被自动初始化为nil,目前还不会引用到`Person`类的实例。

```
var reference1: Person?
var reference2: Person?
var reference3: Person?
```
现在创建Person类的实例

```
reference1 = Person(name: "John Appleseed")
// prints "John Appleseed is being initialized”
```
应当注意到当你调用`Person`类的构造函数的时候，初始化的信息被打印出来。由此可以确定构造函数被执行。

由于`Person`类的新实例被赋值给了`reference1`变量，所以`reference1 `到`Person`类的新实例之间建立一个强引用。正是因为这一个强引用，ARC会保证`Person`实例被保持在内存中不被销毁。

如果你将同一个`Person`实例也赋值给其它两个变量，该实例又会多出两个强引用:

```
reference2 = reference1
reference3 = reference1
```
现在这个 `Person`实例有三个强引用了。

如果你通过给其中两个变量赋值`nil`的方式断开两个强引用，只留下一个强引用，`Person`实例也不会被销毁。

	reference1 = nil
    reference2 = nil
   
 在你清楚地表明不再使用这个Person实例时，第三个也就是最后一个强引用被断开时，ARC会销毁它:
 
 
 	 reference3 = nil
    // 打印 “John Appleseed is being deinitialized”
    

## 类实例之间的循环强引用

在上面的例子中，ARC会跟踪你所新创建的`Person`实例的引用数量，并且会在`Person`实例不再被需要时销毁它。

然而，我们可能会写出一个类实例的强引用永远不能变成0的代码。如果两个类实例互相拥有对方的强引用，因而每个实例都让对方一直存在，就是这种情况。这就是所谓的强引用。

你可以通过定义类之间的关系为弱引用或者无主引用，来避免强引用。不管怎样，让我们先来了解下它是如何产生的。

下面展示一个不经意产生循环强引用的例子。例子定义了两个类:`person`和`Apartment`,用来建模公寓和它其中的居民:

```
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}
```
```
class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}
```

每一个`Person`实例有一个类型为`String`,名字为`name`的属性，并有一个可选的初始化为nil的`apartment`属性。`apartment`属性是可选的，因为一个人并不总是拥有公寓。

类似的，每个`Apartment`实例有个叫`unit`,类型为String的属性，并有一个可选的初始化为nil的`tenant`属性。`tenant`属性是可选的，因为一栋公寓并不总是有居民。

这两个类都定义了析构函数，用以在类实例被析构的时候输出信息。这能让你知道`Person`和`Apartment`的实例是否像预期那样被销毁。

接下来的代码片段中定义了两个可选类型的变量`john`和`unit4A`,并分别被设定为下面的的`Apartment`和`Person`的实例。这两个变量都被初始化为nil:

```
var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")
```
在两个实例被创建和赋值后，下面表现了强引用的关系。变量`john`现在有个指向`Person`实例的强引用，而变量`unit4A`有一个指向`Apartment`实例的强引用:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/referenceCycle01_2x.png)
现在，你能够将两个实例关联在一起了，这样人就能有公寓住了，而工具也有了房客。注意感叹号是用来展开和访问可选变量`john`和`unit4A`中的实例，这样实例的属性才能被赋值:

```
john!.apartment = unit4A
unit4A!.tenant = john
```
在将两个实例联系在一起后，强引用的关系如下:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/referenceCycle02_2x.png)

不幸的是，这两个实例关联后会产生一个循环强引用，`Person`类现在有了一个指向`Apartment`实例的强引用，而`Apartment`实例也有了一个指向`Person`的强引用，因此，当你断开`john`和`uni4A`变量所持有的强引用时，引用计数并不会降到0.实例也不会被销毁。

```
john = nil
unit4A = nil
```

>**注意:**当你把这两个变量设置为nil时，没有任何一个析构函数被调用，循环强引用会一直阻止`Person`和`Apartment`类实例被销毁，这既是你应用程序中造成的内存泄露。



在你将`john`和`unit4A`设置为nil后，强应用关系如下:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/referenceCycle03_2x.png)

Person和Apartment实例之间的强引用关系保留了下来并且不会被断开。


## 解决实例之间的循环强引用

swift提供了两种方法用来解决在你使用类的属性时遇到的循环强引用的问题：弱引用(weak refrence)和无主引用(unowned refrecnce).

弱引用和无主引用允许循环引用中的一个实例引用另外一个实例而不包吃强引用。这样实例能够互相引用而不产生循环强引用。

对于生命周期中会变成nil的实例使用弱引用。相反地，对于初始化赋值后再也不会给赋值为nil的实例，使用无主引用。

###弱引用

弱引用不会对其引用的实例保持强引用，因而不会阻止ARC销毁被引用的实例。这个特性阻止了引用变为循环强引用。声明属性或者变量时，在前面加上`weak`关键字表明这是一个弱引用。

在实例的声明周期中，如果某些时候引用没有值，那么弱引用可以避免循环强引用。如果引用总是有值，则可以使用无主易用。在上面`Apartment`的例子中，一个公寓的生命周期中，有时是没有‘居民’的，因此适合使用弱引用来解决循环强引用。

>> *注意*
>> 弱引用必须被声明为变量，表明其值在运行时会被修改，若引用
>> 不能被声明为常量。
>> 
>> 
>> 
因为弱引用可以没有值，你必须将每一个弱引用声明为可选类型。在Swift中，推荐使用可选类型描述可能没有值的类型。

因为弱引用不会曹植所引用的实例，即使引用存在，实例也有可能被销毁。因此，ARC会在引用的实例被销毁后自动将其赋值为nil.你可以像其他可选值一样，检查弱引用的值是否存在，你讲永远不会访问已销毁的实例的引用。

下面的例子跟上面 `person`和`Apartment`的例子一致，但是有一个重要的区别。这一次，`Apartment`的`tenant`	属性被声明为弱引用:

```
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}
```

```
class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    weak var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}
```
然后跟之前一样，建立两个变量(john和unit4A)之间的强引用，并关联两个实例：

```
var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john
```
现在，两个关联在一起的实例的引用关系如下图所示:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/weakReference01_2x.png)

`Person`实例依然保持对`Apartment`实例的强引用，但是`Apartment`实例只持有对`Person`实例的弱引用。这意味着当你断开`john`变量所保持的强引用时，再也没有指向`Person`实例的强引用了：
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/weakReference02_2x.png)

由于再也没有指向`Person`实例的强引用，该实例会被销毁:

```
john = nil
// 打印 “John Appleseed is being deinitialized”
```
唯一剩下指向`Apartment`实例的强引用来自变量`unit4A`。如果你断开这个强引用，再也没有指向`Apartment`实例的强引用了:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/weakReference03_2x.png)
由于再也没有指向`Apartment`实例的强引用，该实例也会被销毁:

```
unit4A = nil
// 打印 “Apartment 4A is being deinitialized”
```
上面的两段代码展示了变量`john`和`unit4A`在被赋值为`nil`后，`Person`实例和`Apartment`实例的析构函数都打印出了“销毁”的信息。
这证明了引用循环被打破了。

### 无主引用
和弱引用类似，无主引用不会牢牢保持住引用的实例。和弱引用不同的是，无主引用是永远有值的。因此，无主引用总是被定义为非可选类型.你可以在声明属性或者变量时，在前面加上关键字 `unowned`表示这个一个无主引用。

由于无主引用是非可选类型，你不需要再使用它的时候将它展开。无主引用总是可以被直接访问。不过ARC无法再实例被销毁后将无主引用设为`nil`,因为非可选类型的变量不允许被赋值为`nil`.

>**注意:**如果你试图在实例被销毁后，访问该实例的无主引用，会出发运行时错误。使用无主引用，你必须确保引用始终指向一个未销毁的实例。
>还需要注意的是如果你试图访问实例已经被销毁的无主引用，Swift确保程序会直接崩溃，而不会发生无法预期的行为。所以你应当避免这样的事情发生。
>
下面的例子定义两个类，`Customer`和`CrediCard`，模拟了银行客户和客户的信用卡。这两个类中，每一个都将另外一个类的实例作为自身的属性。这种关系可能会造成循环强引用。

`Customer`和`CrediCard`之间的关系与前面若引用例子中`Apartment`和`Person`的关系略为不同。在这个数据模型中，一个客户可能有或者没有信用卡，但是一张信用卡总是关联着一个客户。为了表示这种关系，`Customer`类有一个可选类型的`Card`属性，但是`CreditCard`类有个一非可选类型的`Customer`属性。


此外，只能通过将一个`number`值和`customer`实例传递给`CreditCard`构造函数的方式来创建`CreditCard`实例。这样可以确保当创建`CreditCard`实例时总是有一个`customer`实例与之关联.

由于信用卡总是关联着一个客户，因此将`customer`属性定义为无主引用，用以避免循环强引用:

```
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { print("\(name) is being deinitialized") }
}
```

```
class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { print("Card #\(number) is being deinitialized") }
}
```
下面的代码段定义了一个叫`john`的可选类型`Customer`变量，用来保存某个特定客户的引用。由于是可选类型，所以变量被初始化为`nil`:

```
var john: Customer?
```
现在你可以创建`Customer`类的实例，用它初始化`CreditCard`实例，并将新创建的`CreditCard`实例赋值为客户的`card`属性:

```
john = Customer(name: "John Appleseed")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)
```
在你关联两个实例后，它们的引用关系如下图所示:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/unownedReference01_2x.png)

`Customer`实例持有对`CreditCard`实例的强引用，而`CreditCard`实例持有对`Customer`实例的无主引用。
由于`Customer`的无主引用，当你断开`john`变量持有的强引用时，再也没有指向`Customer`实例的强引用了：
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/unownedReference02_2x.png)

由于再也没有指向`Customer`实例的强引用了，该实例被销毁了，其后，再也没有指向`CreditCard`实例的强引用，该实例也随之被销毁了：

```
john = nil
// 打印 “John Appleseed is being deinitialized”
// 打印 ”Card #1234567890123456 is being deinitialized”
```
最后的代码展示了在`john`变量被设为`nil`后`Customer`实例和`CreditCard`实例的构造函数都打印出了"销毁"的信息.

###无主引用以及隐式解析可选属性
上面弱引用和无主引用的例子涵盖了两种常用的需要打破循环强引用的场景。
`Person`和`Apartment`的例子展示了两个属性的值都允许为`nil`,并会潜在的产生循环强引用。这种场景最适合用弱引用来解决。

`Customer`和`CreditCard`的例子展示了一个属性的值允许为`nil`,而另一个属性的值不允许为`nil`,这也可能会产生循环强引用。这种场景最适合通过无主引用来解决。

然而，存在着第三种场景，在这种场景下，两个属性都必须有值，并且初始化完成永远不会为`nil`.在这种场景下，需要一个类使用无主属性，而另外一个类使用隐式解析可选属性。

这使两个属性在初始化完成后能被直接访问,同时避免了循环了引用。

下面的例子定义了两个类，`Country`和`City`，每个类将另外一个类的实例保存为属性。在这个模型中，每个国家必须有首都，每个城市必须属于一个国家。为了实现这种关系，`Country`类拥有一个`capitalCity`属性，而`City`类有一个`Country`属性:

```
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}
```

```
class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```

为了建了两个类的依赖关系，`City`的构造函数接受一个`Country`实例作为参数，并且将实例保存到`country`属性。

`Country`的构造函数调用了`City`的构造函数。然而，只有`Country`的实例完全初始化后，`Country`的构造函数才能把`self`传给`City`的构造函数。

为了满足这种需求，通过在类型结尾处加上感叹号 （City!） 的方式，将`Country`的`capitalCity`属性声明为隐式解析可选类型的属性。这意味着像其他可选类型一样，`capitalCity`属性的默认值为`nil`,但是不需要展开它的值就能访问它.

由于`capitalCity`默认值是nil,一旦`Country`的实例在构造函数中给`name`属性赋值后，整个初始化过程就完成了。这意味着一旦`name`属性被赋值后，`Country`的构造函数就能引用并传递隐式的`self`。`Country`的构造函数在赋值`capitalCity`时，就能将`self`作为参数传递给`City`的构造函数。

以上的意义在于你可以通过一条语句同时创建`Country`和`City`的实例，而不产生循环强引用，并且`capitalCity`的属性能被直接访问，而不需要通过感叹号来展开它的可选值:

```
var country = Country(name: "Canada", capitalName: "Ottawa")
print("\(country.name)'s capital city is called \(country.capitalCity.name)")
// 打印 “Canada's capital city is called Ottawa”
```
在上面的例子中，使用隐式可选值意味着满足了类的构造函数的两个构造阶段的要求。`capitalCity`属性在初始化完成后，能像非可选值一样使用和存取，同时还避免了循环强引用。


## 闭包引起的循环强引用
前面我们看到了循环强引用是在两个类实例属性互相保持对方的强引用时产生的，还知道了如何用弱引用和无主引用来打破这些强引用。

循环强引用还会发生在你将一个闭包赋值给类实例的某个属性，并且这个闭包体中又使用了这个类实例时。这个闭包体中可能访问了实例的某个属性，例如`self.someProperty`,或者闭包中调用了实例的某个方法，例如：`self.someMethod()`,这两种情况都会导致闭包'捕获'self,从而产生循环强引用。

循环强引用的产生，是闭包和类相似，都是引用类型。当你把一个闭包赋值给某个属性时，你是将这个闭包的引用赋值给了属性，实质上，这跟之前的问题是一样的，两个强引用让彼此一直有效。但是，和两个类实例不同，这次一个是类实例，另一个是闭包。

Swift提供了一种优雅的方法来解决这个问题，称之为`闭包捕获列表`.同样地，在学习如何用闭包捕获列表打破循环强引用之前，先来了解一下这里的循环强引用是如何产生的。

下面的例子为你展示了当一个闭包引用了`self`后是如何产生一个循环强引用的。例子中定义了一个`HtmlElement`的类，用一种简单的模型表示HTML文档中一个单独的元素:

```
class HTMLElement {

    let name: String
    let text: String?

    lazy var asHTML: Void -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\(name) is being deinitialized")
    }

}
```
`HTMLElement`类定义了一个`name`属性来表示这个元素的名字，例如代表段落的`p`,或者代表换行的`br`,`HTMLElement`还定义了一个可选属性`text`,用来设置HTML元素呈现的文本。

除了上面的两个属性，`HTMLElement`还定义了一个lazy属性`asHtml`,这个属性引用了一个将`name`和`text`组合成html字符串片段的闭包。该属性是`Void->String`类型，或者理解为'一个没有参数，返回`String`的函数'。

默认情况下，闭包赋值给了`asHtml`属性，这个闭包返回一个代表HTml标签的字符串。如果`text`值存在，该标签就包含`text`;如果不包含`text`,该标签就不包含文本，对于段落元素，根据`text`是否有值，闭包会返回`"<p>some text</p>"`或者`"<p />"`

可以像实例方法那样去命名，使用`asHtml`属性。然而，由于`asHtml`是闭包而不是实例方法，如果你想改变特定html元素处理方式的话，可以使用自定义闭包来取代默认值。

例如，可以将一个闭包赋值给`asHtml` 属性，这个闭包能在`text`属性是`nil`时使用默认文本，这是为了避免返回一个空的HTML标签:

```
let heading = HTMLElement(name: "h1")
let defaultText = "some default text"
heading.asHTML = {
    return "<\(heading.name)>\(heading.text ?? defaultText)</\(heading.name)>"
}
print(heading.asHTML())
// 打印 “<h1>some default text</h1>”
```

>**注意**
>asHtml声明为lazy属性，因为只有当元素确实需要被处理为HTML输出的字符串时，才需要使用`asHTML`.也就是说，在默认的闭包中可以使用`self`,
>因为只有初始化完成以及`self`确实存在后，才能访问lazy属性。



HTMLElement类只提供了构造函数，通过`name`和`text`参数来初始化一个新元素，该类也定义了一个析构函数，当 `HTMLElement`被销毁时，打印一条消息

```
var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// 打印 “<p>hello, world</p>”
```

>**注意**
>上面的paragraph是可选类型，因此我们可以赋值`nil`来演示循环强引用
>


不幸的是，上面写的 `HTMLElement`类产生了类实例和作为`asHtml`默认是和闭包之间产生了循环强引用，循环强引用如下图所示:

![lgoo](http://7xkxhx.com1.z0.glb.clouddn.com/closureReferenceCycle01_2x.png)

实例的`asHTML`属性持有闭包的强引用。但是，闭包在其体内使用了`self`(引用了`self.name`和`self.text`),因此捕获了`self`,这意味着闭包又反过来持有了`HtmlElement`实例的强引用。这样两个对象就产生了循环强引用。

>**注意**
>随让闭包多次使用了`self`,它只捕获`HTMLElemnt`实例的一个强引用。
>

如果设置`paragraph`变量为`nil`,打破它持有的`HTMLElement`实例的强引用，`HTMlElement`实例和它的闭包都不会被销毁，也是因为循环强引用：

	paragraph = nil
注意,`HTMLElement`的析构函数并没有被打印，证明了 `HTMLElement`实例并没有被销毁.



## 解决闭包引起的循环强引用

在定义闭包时同事定义捕获列表作为闭包的一部分，通过这种方式可以解决闭包和类之间的循环强引用。捕获列表定义了闭包体内捕获了一个或者多个引用类型的规则。跟解决两个类实例间的循环强引用一样，声明每个捕获的引用为弱引用或无主引用，而不是强引用。应当根据代码关系来决定使用弱引用还是无主引用。

>**注意**
>Swift有如下要求:只要在闭包内使用`self`的成员，就要用`self.someProperty`或者`self.someMethod()`,（而不只是`someProperty,someMethod()`）,这提醒你可能会一不小心就捕获了`self`
>
>


###定义捕获列表
捕获列表中的每一项都由一对元素组成，一个元素是`weak`或者`unowned`关键字，另一个元素是类实例的引用(例如self)或初始化过的变量(如 delegate=self.delegate!)

如果闭包有参数列表和返回类型，把捕获列表放在它们前面:

```
lazy var someClosure: (Int, String) -> String = {
    [unowned self, weak delegate = self.delegate!] (index: Int, stringToProcess: String) -> String in
    // 这里是闭包的函数体
}
```

如果闭包没有指明参数列表或者返回类型，即它们会通过上下文判断，那么可以把捕获列表和关键字 `in`放在闭包最开始的地方:

```
lazy var someClosure: Void -> String = {
    [unowned self, weak delegate = self.delegate!] in
    // 这里是闭包的函数体
}
```

###弱引用和无主引用

在闭包和捕获的实例总是互相引用并且总是同时销毁时，将闭包内的捕获定义为`无主引用`.

相反的，在被捕获的引用可能会变成`nil`时，将闭包内的捕获定义为`弱引用`.弱引用总是可选类型，并且当引用的类型被销毁后，弱引用的值会自动设置为`nil`.这使我们可以在闭包体内简单它们是否存在.

>**注意**
>如果被捕获的引用绝对不会变成`nil`,应该用无主引用，而不是弱引用
>

前面的`HTMLElement`例子中，无主引用是正确的解决循环强引用的方法，这样编写`HTMlElement`来避免循环强引用:

```
class HTMLElement {

    let name: String
    let text: String?

    lazy var asHTML: Void -> String = {
        [unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\(name) is being deinitialized")
    }

}
```

上面的`HTMLElement`实现和之前的实现一致，除了在`asHTML`闭包中多了一个捕获列表。这里捕获列表是`[unowded self]`,表示将`self`捕获为无主引用而不是强引用.


```
var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// 打印 “<p>hello, world</p>”
```

使用捕获列表后引用关系如下图所示:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/closureReferenceCycle02_2x.png)

这一次，闭包以无主引用的形式捕获`self`,并不会持有`HTMLElement`实例的强引用。如果将`paragraph`赋值为nil,`HTMLElement`实例将会被销毁，并能看到析构函数打印出消息;

```
paragraph = nil
// 打印 “p is being deinitialized”
```







