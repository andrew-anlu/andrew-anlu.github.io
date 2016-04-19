---
layout: post
title: "swift基础知识"
date: 2016-02-26 14:37:36 +0800
comments: true
categories: swift
---
swift充分利用了语言设计的悠久历史，拥有非常多的设计特性，使得软件开发变得更容易，更简单，更安全。

* 安全

swift设计的初衷就是一门安全的语言，c语言中有许多缺陷，比如意外使用null指针，这些很难在swift中遇到。swift非常重视强类型化，除了一些极为特殊的情况下，它是不允许为空的。
<!--more-->

* 现代

swift包含了大量的现代语言的特性，可以轻松的表达代码逻辑。这些特性包括：模式匹配switch语句，闭包，所有值都是对象的概念

* 强大

swift可以访问整个Object-c运行时，而且可以无缝的连接到Object-c的类，意味着我们可以马上用swift编写出完整的ios和OSX App,不用等着别人从Oc向swift移植任何功能

##基础语法

###变量和常量
在swift中，let用来定义常量，如果一个值定义之后不希望再被改变，可以用let来声明

var用来定义变量，如果一个值一直再被赋值和变化，则可以用var来声明

swift中的常量必须拥有值，如果定义了一个常量，但是没有给定值会报错的。

```
let someConstant : Int
//错误，常量在声明时必须包含值
```

变量可以不包含值，只要不尝试访问它就行。换句话说，如果创建了一个变量，但没有为它设置值，那唯一能做的就是为它指定一个值。之后就可以使用它了。

```
var someVariable:Int
someVariable += 2
//错误，someVariable没有包含值，所以不能向它增加2

someVariable=2
someVariable+=2
//成功，因为someVariable有一个值，可以进行平常操作

```

###类型
我们不需要定义一个变量是什么类型，Swift可以根据它的初始值做出判断。这意味着在你定义一个变量并为其设定数值2时，这个变量会是Int类型：

	//隐式指定整数类型
	var anInteger = 2
	
大多数类型都不能合并，因为编译器不知道结果会是什么类型。例如：我们不能将一个string值加到一个int值上。因为其结果毫无意义。


在Object-c中，nil实际上被定义为一个指向0的void指针。严格来说，它是一个数字，这就意味着我们可以进行类似下面的操作：
	
	int i=(int)(nil)+2
	//等于2（因为0+2=2）
	
这在swift中是不允许的，因为nil和Int是不同类型

swift中的所有变量都是需要有取值的。如果希望允许一个变量在某些时候为nil,那就使它成为一个可选变量。可选变量的定义是在其类型中包含一个问号(?)

	//可选整数，允许为nil
	var anOptionalInteger : Int? = nil
	anOptionalInteger = 22

只有可选变量才允许被设置为nil.如果一个变量没有被定义为nil,那就不允许将它设定为nil值:
	
	//非可选，不允许为nil
	var aNonOptionalInteger = 32
	
	aNonOptionalInteger  = nil
	//错误，只有可选值才能为nil

可以使用if语句来查看一个可选变量是否拥有值
	
	if anOptionalInteger !=nil {
	   println("It has a value")
	}else{
	   println("It has no value");
	}

对于可选变量，可以进行拆包操作，获得其取值。这一工作用 感叹号 !实现。

#####注意
如果对一个可选变量进行拆包，而它并没有值，程序将会抛出一个运行时错误，并会崩溃

	//可选类型必须使用!拆包
	anOptionalInteger=2
	1+anOptionalIntger!  //3
	
	anOptionalInteger = nil
	1+anOptionalInteger!
	//崩溃：anOptionalInteger=nil,不能使用nil数据
	
如果不希望每次用到可选变量都要对其进行拆包，可以将它声明为已拆包的
	
	var unwrappedOptionalInteger:int!
	unwarappedOptionalInteger=1
	1+ unwarappedOptionalInteger  //2
这样就可以直接使用它们的值，但可能会不安全（因为它让你逃避了在需要时对其进行拆包的操作，可能会让你忘了它们有时会是nil）.谨慎使用

在swift中，可以在不同类型之间进行转换。例如：要将一个Int转换为一个string,可以这样做
	
	let aString=String(anInteger)
	//"2"
	
###元组
元组是数据的一个简单集合。利用元组，可以将多个值一起捆绑到单个值中：

	let aTuple = (1,"YES")
有了元组，就可以从中提取到值：
	
	let theNumber = aTuple.0  //=1
	
除了用数字提取元组的值之外，还可以为元组中的值添加标签：

	let anotherTuple = (aNumber：1,aString: "YES")
	let theOhterNumber = anotherTuple.aNumber //1
###数组
swift中的数组很容易使用。要创建一个数组，可以直接用 []:

	//整数数组
	let arrayOfInteger : [Int] = [1,2,3]
	
Swift还可以推断出数组的类型：
	
	//暗含了数组类型
	let implicitArrayofIntegers=[1,2,3]
还可以创建空数组，不过，这种情况需要人工指定其类型
	
	let anotherArray=[Int]()

用let关键字定义的数组，其内容是不可变的，也就是说，不允许改变其内容

有了数组之后，就可以使用其内容了。例如，可以使用append函数向数组的末尾追加对象

	var myArray =[1,2,3,4]
	myArray.append(4)
除了在数组的末尾追加对象之外，还可以插入对象
	
	myArray.insert(5,atIndex:0)
###字典
字典是一种将键映射到值的类型。当希望表示一组相关信息时，字典是很有用的。
声明一个字典
	
	var crew=["caption":"张三",
	         "first officer":"jack",
	         "second officer":"david"]	
	       
有了字典，就可以通过下标来访问其内容。下标就是在变量名之后用方括号 []的地方。
	
	crew["Caption"]   //=张三
	
### 控制流
在swift中，所有的if语句以及所有的循环的主体都需要放在两个大括号。

	if(something){
	 //这对大括号是必须的
	}
当拥有一个集合时，比如一个数组，可以使用for-in循环来迭代每一项:
	
	let loopingArray=[1,2,3,4,5]
	var loopSum=0
	for number in loopingArray{
	   loopSum += number
	}
还可以使用for-in循环来迭代一个数值范围。例如

	var firstCount = 0
	for index in 1..<10{
	  firstCounter++
	}	
	//循环9次

注意第一行中的..<运算符，这是一个范围运算符，swift用它描述一个值到另一个值的数值范围。实际上有两个范围的运算符:两个句点加一个做尖括号(..<),表示从第一个值开始知道最后一个值的一个范围（最后一个值不包含在内），例如5..<9 包含了数字 5,6,7,8. 如果希望创建一个包含最后数值的范围，可以使用三个句点(...),这里没有尖括号，范围5...9包含的数值 5,6,7,8,9

###Switch
switch是一种根据变量值运行代码的强大方式。
 
switch可以判断整数，还可以判断字符串

	let stringSwitch="Hello"
	switch stringSwitch{
	  case "Hello":
	     println("A greeting");
	  case "Goodbye":
	  	 println("A farewell")
	  default:
	  	println("something");
	  	
	}
还可以对元组进行判断
	
	var str = "Hello, playground"

	let tupleSwith=("YES",123)
	switch tupleSwith{
	case ("YES",123):
	    print("Tuple contains 'YES' and 123");
	    break;
	case ("YES", 1):
	    print("Tuple contains 'YES' and else")
	    break;
	default:
	    print("tuple something")
	    break;
	}
switch的工作方式与C和Object-C中有点不同，在swift中，switch语句中的某一部分执行完毕后，不会自动“落入”下一部分，也就是说，不需要使用break关键字。

###函数和闭包
函数可以向调用它们的代码返回一个值。在定义一个具有返回值的函数时，必须使用箭头(->)指明所返回数据的类型
	
	func sayHello()->Int{
	 return 123
	}
	sayHello()
	
我们也可以向函数中传送参数，让其能够利用它们完成任务，在为函数定义参数时，还需要定义这些参数的类型：

	func thirdFunction(firstValue:Int, secondValue:Int)->Int{
	  return firstValue+secondValue;
	}
	thirdFunction(1,2)
	
一个函数只可以返回一个值，我们前面已经看到这种情况，但也可以通过元组的方式返回多个值。另外，可以为元组中的值附加名字，以便能够更轻松的处理返回值：
	
	func fourthFunction(firstValue:Int,secondValue:Int)->(doubled:Int,test:Int){
		return (firstValue+1,secondValue*4)
	}
	fourthFunction(2,3)

在调用一个会返回元组的函数时，可以用数字访问它的值，如果有名字的话，也可以用名字来访问：

	//用数字访问
	fourthFunction(2,4).1 //16
	//其他相同，只是使用了名字:
	
	fourthFunction（2，4）.test //16

在定义函数时，可以为参数指定名字，当无法马上明白每个参数的用途时，这一功能会非常有用。可以像下面这样来定义参数的名字

	func addNumbers(firstNumber num1:Int,toSecondNumber   num2:Int)->Int{
    return num1+num2;
    }
    addNumbers(firstNumber: 2, toSecondNumber: 3) //5 
  
 在为参数创建名字时，就是为参数创建一个内部名字和一个外部名字。内部名字供函数引用该参数，而外部名字工调用该函数的外部代码使用。如果函数没有命名参数，那每个参数就只有一个内部名字。
 
 
在创建参数时，还可以为其参数指定默认值，这就意味着在调用这些函数时可以省略特定的参数；

	 
    func multiplyNumber(firstNumber:Int,multiplier:Int=2)->Int{
    return firstNumber+multiplier;
    }
    //可以省略具有默认值的参数
    multiplyNumber(2);//4

有时，我们希望使用参数个数可变的函数，一个取值数目可变的参数成为可变参数，在这写情况下，我们希望一个函数能够处理任意数目的参，从0到一个无限数，为此可以使用三个句点(...)表示一个参数的取值是可变的。在函数的主体内部，可变参数变成一个数组，我们可以像使用其他数组一样使用它。

```
func sumNumbers(numbers:Int...)->Int{
  //在这个函数中，'numbers'是一个数组
    var total=0;
    for number in numbers{
        total+=number;
    }
    return total;
}

sumNumbers(2,2,3,4) //11
```
通常，函数以参数为输入时是按值传递的，输出返回的也是值，但是，如果有用inout关键字定义一个参数，那就可以按引用传送改参数，直接改变这个变量中存储的值。采用这种方式，可以用一个函数交换两个变量，如下所示:

```
	func swapValues(inout firstValue:Int,inout secondValue:Int){
    let tempValue=firstValue;
    firstValue=secondValue;
    secondValue=tempValue;
}
var swap1=2;
var swap2=3;
swapValues(&swap1, secondValue: &swap2)
print(swap1) //3
print(swap2) //2
```
在调用函数时，这个变量的值可能会发生变化。

###将函数作为变量
函数可以存储在变量在中，为此，首先声明一个变量，它能够存储一个接受特定参数，返回一个值的函数。声明之后，只要一个函数的参数与返回值类型都与声明中的函数相同，可以将它存储在这个变量中：

```
func test1(v1:Int,v2:Int)->Int{
     return v1+v2
 }
  var numbersFunc:(Int,Int)->Int;
  //numbersFunc现在可以存储任何接受两个int并返回一个int的函数
  numbersFunc=test1;
  numbersFunc(1,2) //3
```
函数还可以接受其他函数作为参数并使用它们。这意味着可以将函数合并到一起。

```
func timesThree(number:Int)->Int{
    return number*3;
}
func doSomethingToNumber(anumber:Int,thingToDo:(Int)->Int)->Int{
//我们已经接受了某一函数作为参数，在本函数中将其称为"thingToDo"
    
    //使用‘anumber’作为参数调用函数 thingToDo
    return thingToDo(anumber);
}

doSomethingToNumber(100, thingToDo: timesThree); //300
```

函数还可以返回其他函数。这意味着可以用函数创建新函数，并在自己的代码中使用这个新函数：

```
//返回一个函数
func createAdder(numberToAdd:Int)->(Int)->Int{
    func adder(number:Int)->Int{
      return number+numberToAdd
    }
    return adder;
}

var addTwo=createAdder(12)
addTwo(12)  //24
```
###闭包

swift的另一个特性是闭包-一些小的匿名代码块，可以像函数一样使用。可以非常方便的将闭包传送给其它函数，告诉他们应当如何执行某一任务。


```
 * 全局函数和嵌套函数其实就是特殊的闭包。 
 * 闭包的形式有： 
 * （1）全局函数都是闭包，有名字但不能捕获任何值。 
 * （2）嵌套函数都是闭包，且有名字，也能捕获封闭函数内的值。 
 * （3）闭包表达式都是无名闭包，使用轻量级语法，可以根据上下文环境捕获值。 
 *  
 * Swift中的闭包有很多优化的地方: 
 * (1)根据上下文推断参数和返回值类型 
 * (2)从单行表达式闭包中隐式返回（也就是闭包体只有一行代码，可以省略return） 
 * (3)可以使用简化参数名，如$0, $1(从0开始，表示第i个参数...) 
 * (4)提供了尾随闭包语法(Trailing closure syntax)
```


为了举例说明闭包如何工作，请考虑内置的sorted函数。这个函数接受一个数组和一个闭包作为参数，并用这个闭包来确定应当如何对数组的各个元素进行排序。

sorted函数接受一个数组，并返回同一数组的一个有序版本。除了sorted函数之外，有一个sort函数，它接受一个数组，并将其修改为有序版本:

	var sortingInline=[2,3,5,19,1,10]
	sort(&sortingInline)
	sortingInline //
	
要对一个数组进行排序，使小的数字出现在大数字之前，可以这样做：
	
	var numbers=[2,3,5,19,1,10]
	var numbersSorted=sorted(numbers,{(n1:Int,n2:Int) - >Bool in 
	//进行排序，使得小数字出现在大数字之前
		return n2>n1;
	})
和函数一样，闭包可以接受参数。在上面的例子中，闭包指定了它所处理参数的名字和类型。但是，并不需要写的特别详细，编译器可以替我们推断参数的类型，非常类似于推断变量类型的方式。

	var numbersSortedReverse=sorted(numbers,{n1,n2 in 
	     return n1>n2;
	})
如果不是特别在意参数拥有什么样的名字，可以让它更简便一些。如果省略了参数名，可以直接根据数字来引用每个参数（第一个参数名称为$0,第二个为$1）

另外，如果闭包只包含一行代码，可以省略return 关键字
	
	var numbersSortedAgain = sorted(numbers,{
	       $1>$0
	})
如果一个闭包是函数调用中的最后一个参数，可以将它放在括号外面。这纯粹是为了提高可读性，不会改变闭包的工作方式


	var numbersSortedReversedAgain=sorted(numbers){
	   $0>$1
	}
	

```
/* 
 * 如果函数需要一个闭包参数作为参数，且这个参数是最后一个参数，而这个闭包表达式又很长时， 
 * 使用尾随闭包是很有用的。尾随闭包可以放在函数参数列表外，也就是括号外。如果函数只有一个参数， 
 * 那么可以把括号()省略掉，后面直接跟着闭包。 
 */   
// Array的方法map()就需要一个闭包作为参数  
let strings = numbers.map { // map函数后面的()可以省略掉  
  (var number) -> String in  
  var output = ""  
  while number > 0 {  
    output = String(number % 10) + output   
    number /= 10  
  }  
  return output  
}  
      
/* 捕获值 
 * 闭包可以根据环境上下文捕获到定义的常量和变量。闭包可以引用和修改这些捕获到的常量和变量， 
 * 就算在原来的范围内定义为常量或者变量已经不再存在（很牛逼）。 
 * 在Swift中闭包的最简单形式是嵌套函数。 
 */   
func increment(#amount: Int) -> (() -> Int) {  
  var total = 0  
  func incrementAmount() -> Int {  
    total += amount // total是外部函数体内的变量，这里是可以捕获到的  
    return total  
  }  
  return incrementAmount // 返回的是一个嵌套函数（闭包）  
}     
      
// 闭包是引用类型，所以incrementByTen声明为常量也可以修改total  
let incrementByTen = increment(amount: 10)   
incrementByTen() // return 10,incrementByTen是一个闭包  
// 这里是没有改变对increment的引用，所以会保存之前的值  
incrementByTen() // return 20     
incrementByTen() // return 30     
  
let incrementByOne = increment(amount: 1)  
incrementByOne() // return 1  
incrementByOne() // return 2      
incrementByTen() // return 40   
incrementByOne() // return 3 
```

###对象
在swift中类看起来是这样的
	
	  class Vechicle {
        var color:String?
        var maxSpeed=80
        
        func description()->String{
          return "A\(self.color) vehicle";
        }
        func travel(){
           print("Traveling at \(maxSpeed) kph")
        }
    }
  类中既包含了属性也包含方法。属性和方法都是类的组成部分，属性是变量，方法是函数。
  
  例如，要定义Vehicle类的一个实例，我们定义一个变量
  	
  	var redVehicle=Vehicle()
  	redVehicle.color="Red";
  	redVehicle.maxSpeed=100;
  	redVehicle.travel();
  	redVehicle.description()
  	
### 初始化与反初始化
在swift中创建对象时，会调用一个被称为其初始化器的特殊方法。初始化器是用来为对象设定初始状态的方法。

除了初始化器之外，还有一个反初始化器，可以在对象消失时运行其中的代码。此方法在对象的retaincount数降到0时运行，就在要将该对象从内存中清除时调动。要想在对象永远消失之前进行一些必要的清理工作，这是最后一个机会。

```
class InitAndDeinitExample {
    //指定的初始化器(也就是主初始化器)
    init(){
     print("I've been creted")
    }
    //便捷初始化器，是调用上述指定初始化器所必须的
    convenience init(text:String){
        self.init();//这是必须的
        print("I was called with the convenience initializer!")
    }
    
    //反初始化器
    deinit{
      print("I'm goint away!")
    }
}
//声明为可选类型
var example:InitAndDeinitExample?
//使用指定的初始化器
example=InitAndDeinitExample()//I've been creted
example=nil //I'm goint away!
//使用便捷初始化器
example=InitAndDeinitExample(text: "Hello"); //I was called with the convenience initializer!
```
初始化器还可以返回nil.当初始化其不能成功地构造一个函数时，这一点很有用。例如，NSURL类有一个初始化器，它接受一个字符串，并将它转化为URL;如果这个字符串不是一个有效的URL,则初始化器返回nil.

要创建一个可以返回nil的初始化器，就在init关键字后面加一个问号，并在初始化器确定它不能成功地构造该对象时，return nil;

		
		convenience init?(value:Int){
    
        self.init();
        if value > 5{
            //不能初始化这个对象;返回nil,表示初始化失败
          return nil
        }
    }
###属性
类将其数据存储在属性中。在前面的例子中，属性是存储在对象中的一个简单值。在swift中，它称为存储属性。但是利用属性可以做很多事情，包括创建一些属性，利用代码来计算它们的值。这些属性称为计算属性，可以用它们提供一个更简单的接口，用来访问类中存储的信息。


例如，考虑一个代表矩形的类，它有一个width属性和一个height属性。再增加一个包含面积值的属性应该是很有用的，但你不想再有第三个属性，而是使用一个计算属性。从外部看来，这就是一个普通属性，但是从内部来说，它实际上是一个函数，可以在需要的时候计算其取值。


要定义一个计算属性，可以像声明存储属性一样声明一个变量，但在后面增加大括号({和})，在这些大括号内部，提供一个get部分，还可以提供有一个set部分；

	class Retangle {
    var width:Double=0.0
    var height:Double=0.0
    
    var area:Double{
      //计算getter
        get{
            return width*height;
        }
        //计算setter
        set{
            width=sqrt(newValue)
            height=sqrt(newValue)
        }
    }
    }
  在上面的例子中，面积是通过计算长和宽的乘积而得到的，这个属性也是可设定的--如果设定了矩形的面积，代码就假定你希望创建一个正方形，并更新宽度和长度值。
  
```
var rect=Retangle()
rect.width=3.0
rect.height=4.5
rect.area
rect.area=9
```
在使用属性时，经常希望在一个属性发生变化时运行某些代码。为支持这一个功能，Swift属性允许向属性添加观察期，也就是一些小的代码块，可以在一个属性值即将发生变化之前运行。要创建一个属性观察期，需在属性后面添加一对大括号，并包含willSet和didSet代码块。这些块会分别获得一个参数-willSet在属性值发生变化之前被调用，它获得的是一个将要设定的值，而didSet获取的是一个旧值:

```
class PropertyObserverExample {
    var number:Int=0{
        willSet(newNumber){
           print("About to change to \(newNumber)")
        }
        didSet(oldNumber){
          print("Just changed from \(oldNumber) to \(self.number)")
        }
    }
    
}

var observer=PropertyObserverExample();
observer.number=4
```
我们还可以让属性变成惰性的。惰性属性就是知道首次访问时才会设定的属性。类的某些设置工作需要耗费大量的时间，利用惰性属性可以将这些工作推迟到将来真正需要时完成。为将一个属性定义为惰性的，可以在它的前面放一个lazy关键字。

```
class SomeExpensiveClass {
    init(id:Int){
     print("Expensive class\(id) created")
    }
}

class LazyPropertyExample {
    var expensiveClass1=SomeExpensiveClass(id:1)
    //注意，我们实际上正在构造一个类，但它被标记为惰性的
    lazy var expensiveClass2=SomeExpensiveClass(id: 2)
    
    init(){
        print("First class created!")
    }
}

var lazyExample=LazyPropertyExample()
//输出"Expensive class1 created",然后输出"First class created"

lazyExample.expensiveClass1 //不输出任何内容，它已经被创建
lazyExample.expensiveClass2 //输出"Expensive class2 created!"
```
在这个例子中，当创建lazyExample变量时，它立即创建SomeExpensiveClass的第一个实例。但是，第二个实例将一直等到代码中实际用到它时才会创建。

###协议
协议可以看做是一个类的需求清单。定义协议，就是创建一个属性和方法清单，类可以声明他们拥有这些属性和方法。

协议看起来与类非常相似，只是我们没有提供任何实际的代码--只是定义了存在哪些种类的属性和函数，以及如何访问它们。

例如：

	protocol Blinking{
    //这个属性必须（至少是）可获取的
    var isBlining:Bool{get}
    
    //这个属性必须是可获取的和可设置的
    var blinkSpeed:Double{get set}
    
    //这个函数必须存在，但是它做些什么由是实现者决定
    func startBlinking(blinkSpeed:Double)->Void
    
}

有了协议，就可以创建遵守协议的类。当一个类遵守了协议时，就是向编译器做出了承诺：它实现了这个协议中列出的所有属性和方法，除此之外，它还可以有其他很多属性和方法，也可以遵守多个协议。
	
	class Light:Blinking {
    var isBlining:Bool=false;
    var blinkSpeed:Double=0.0
    
    func startBlinking(blinkSpeed: Double) {
        print("I am now blikning")
        isBlining=true
        //我们说这里的self.blinkSpeed是为了帮助编译器
        //判断参数'blinkSpeed'和属性之间的区别
        self.blinkSpeed=blinkSpeed;
    }
    }  

调用如下:

```

//可以是任何具有Blinking协议的对象
var aBlinkThing:Blinking?

aBlinkThing=Light();

aBlinkThing!.startBlinking(4.0)

aBlinkThing?.blinkSpeed
```
	
###泛型
Swift是一种静态类型化语言，这就是说，Swift编译器需要确切的了解你的代码正在处理什么类型的信息。这意味着你不能将字符串传送给打算用来出来日期的代码，而在Objective-c中是可能发生这种情况的。

利用泛型，在编写代码时，不需要准确地知道这些信息的类型。数组就是泛型的一个应用实例：数组实际上并没有对自己存储的数据进行任何操作，只是将他们存储为一个有序集合，事实上，数组就是泛型。

要创建一个泛型类型，可以像通常一样为对象命名，然后在两个尖括号之间指定任务泛型类型。传统上使用的术语是T.

	class Tree<T> {
    var value:T
    var children:[Tree<T>]=[]
    
    init(value:T){
        self.value=value;
    }
    
    func addChild(value:T)->Tree<T>{
      let newChild=Tree<T>(value: value)
        children.append(newChild)
        return newChild;
    }
    }

一旦定义了一个泛型，就可以由它创建一个具体的非泛型类型。例如，可以使用刚刚创建的Tree类型，创建一个用于处理Int的版本和一个处理String的版本。

```
//整数树
let integerTree=Tree<Int>(value: 5)

integerTree.addChild(10)
integerTree.addChild(20)

//字符串树
let stringTree=Tree<String>(value: "hello")
stringTree.addChild("YES")
stringTree.addChild("swift")
```

###序列化与反序列化
我们还可以将对象转化为数据。为此，首先要使对象遵守NSObject和NSCoding协议，然后添加两个方法：encodeWithCoder,一个以NScoder为参数的初始化函数：
	
	class SerializableObject:NSObject,NSCoding {
    var name:String?
    func encodeWithCoder(aCoder: NSCoder) {
        aCoder.encodeObject(name!,forKey: "name")
    }
    
    override init(){
       self.name="My object"
    }
    required init(coder aDecoder:NSCoder){
       self.name=aDecoder.decodeObjectForKey("name") as? String
    }
    }
   
  这些对象与数据之间的相互转换非常简单
  
 ```
 let anObject=SerializableObject()

anObject.name="My thing that I'm saving"

//将它转换为数据
let objectConvertedToData=NSKeyedArchiver.archivedDataWithRootObject(anObject);

//将其转换回来，
//注意，此转换可能会失败，所以'unarchiveObjectWithData'返回一个可选值
//因此，使用‘as?’来查看它是否成功

let loadedObject=NSKeyedUnarchiver.unarchiveObjectWithData(objectConvertedToData) as? SerializableObject

loadedObject?.name
 ```
  

	
	
	
  	