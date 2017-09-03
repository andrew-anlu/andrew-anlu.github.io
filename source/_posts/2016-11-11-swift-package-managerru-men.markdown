---
layout: post
title: "swift Package Manager入门"
date: 2016-11-11 14:09:18 +0800
comments: true
categories: swift
---

大部分语言都有官方的代码分配解决方案，幸好苹果也在开发替代[Cocoapods](https://cocoapods.org/)和[Carthage](https://github.com/Carthage/Carthage)的管理工具，[Swift Package Manager](https://swift.org/package-manager/#conceptual-overview)(Swift包管理器，下面我们简称SPM)就是一个用来管理Swift代码的分配的官方工具，它为Swift编译系统集成了自动进行下载，编译和连接依赖的过程

目前，SPM还处于早起阶段，现在仅仅支持OS X和linux系统，尚不支持Ios,watchOS以及tvOS平台，但未来很大希望会支持上述平台。


## 概念概述
在swift中我们使用模块来管理代码，每个模块指定一个命名空间并强制指定模块外那些部分是代码是可以被访问控制的。

一个程序可以将它所有代码聚合到一个模块中，也可以将它作为依赖关系导入到其他模块，除了少量系统提供的模块，像OS X中的Darwin或者 Linux中的Glibc等大多数依赖需要代码被下载或者内置才能被使用。

当你将编写额解决待定问题的代码独立成一个模块时，这段代码可以在其他情况下呗重新利用。例如，一个模块提供了发起网络请求的功能，在一个照片分享的app或者一个天气的app里它都是可以使用的。使用模块可以让你的代码建立在其他开发者的代码之上，而不是你自己去重复实现相同的功能。

一个包由Swift源文件和一个清单文件组成，这个清单文件称为`Package.swift`,定义包或者它的内容使用`PackageDescription `模块。

一个包邮一个或者多个目标，每个目标制定一个铲平并且可能声明一个后者多个依赖。

一个目标可能构建一个库或者一个可执行文件作为其产品。库是包含可以被其它Swift代码导入的模块。可执行文件是一段可以被操作系统运行的程序

目标依赖是指保重代码必须添加的模块。依赖由包资源的绝对或者相对URL和一些可以被使用的包的版本要求所组成。包管理器的作用是通过自动为工程下载和编译所有依赖的过程中，减少协调的成本。这是一个递归的过程：依赖能有自己的依赖，其中每一个也可以具有依赖，形成一个依赖的相关图。包管理器下载和编译所需要满足整个依赖相关图的一切。


## 开源Swift入门

* [下载和安装Swift](#下载和安装Swift)
* [使用REPL](#使用REPL)
* [使用编译系统](#使用编译系统)
* [使用LLDB调试器](#使用LLDB调试器)

关于使用REPL和LLDB调试器的内容具体可以参阅官方文档[使用REPL](https://swift.org/getting-started/#using-the-repl)和[使用LLDB调试器](https://swift.org/getting-started/#using-the-lldb-debugger)


## <a name = "下载和安装Swift"></a>下载和安装Swift

刚开始下载和安装swift需要下载并安装编译器和其它必备组件，进入到 [https://swift.org/download/#releases](https://swift.org/download/#releases)按目标平台的说明进行。


下载完成后，点击按步骤安装就可以

在OS X上下载工具链的默认地址是:`/Library/Developer/Toolchains`.接着，我们可以输入以下命令导出编译路径:

```
$ export PATH=/Library/Developer/Toolchains/swift-latest.xctoolchain/usr/bin:"${PATH}"
```

首先需要安装clang:

```
$ sudo apt-get install clang
```

如果你在Linux上安装的Swift工具链在系统根目录以外的目录，你需要使用你安装Swift的实际路径来运行下面的命令：

```
$ export PATH=/path/to/Swift/usr/bin:"${PATH}"
```

导出路径之后，你可以通过输入 swift 命令并传入 --version 标志来校验你是否运行了 Swift 的预期版本

```
$ swift --version
Apple Swift version 3.0-dev (LLVM ..., Clang ..., Swift ...)
```

在版本号的后缀 -dev 用来表明它是一个开发的编译，而不是一个发布的版本

## <a name = "使用REPL"></a>使用REPL

## <a name = "使用编译系统"></a>使用编译系统
Swift编译系统为编译库，可执行文件和不同工程之间共享代码提供了基本的约定。

创建一个新的Swift包，首先创建并进入到一个新的目录命令为Hello:

```
$ mkdir Hello
$ cd Hello
```

每个包在其根目录下都必须拥有一个命名为`Package.swift`清单文件，如果清单文件为空，那包管理器将会使用常规默认的方式来编译包，创建一个空的清空文件使用命令:

```
$ touch Package.swift
```
当使用默认方式时，包管理器预计将包含在Source/子目录下的所有源代码。创建方式：

```
$ mkdir Sources
```

### 编译可执行文件

默认方式下，目录中包含一个文件称为`main.swift`将会将文件编译成与包名称相同的二进制可执行文件。

在这个例子中，包将生成一个可以输出`hello world`的可执行文件为 *hello*

在*Source/*目录下创建一个命名为`main.swift`的文件，并使用你喜欢的任意一种编译器输入如下代码:

```
print("Hello, world!")
```

返回到 Hello 目录中，通过运行 swift build 命令来编译包：

```
$ swift build
```

当命令完成之后，编译产品将会出现在 .build 目录中。通过如下命令运行 Hello 程序:

```
$ .build/debug/Hello
Hello, world!
```

下一步，让我们在新的资源文件里定义一个新的方法 `sayHello(_:)`然后直接用`print(_:)`替换执行调用的内容。

### 多了源文件协作

在`Sources/`目录下创建一个新文件命名为`Greeter.swift`然后输入如下代码:

```
func sayHello(name: String) {
	print("Hello, \(name)!")
}
```

`sayHello(_:)`方法带一个单一的字符串参数，然后在前面打印一个"hello",后面跟着函数参数单词"World".

现在打开`main.swift`，然后替换原来的内容为下面代码:

```
if Process.arguments.count != 2 {
    print("Usage: hello NAME")
} else {
    let name = Process.arguments[1]
    sayHello(name)
}
```

跟之前的硬编码不同，`main.swift`现在从命令行参数中读取。替代之前直接调用`print(_:)`，`main.swift`现在调用`sayHello(_:)`方法，因为这个方法是`Hello `模块的一部分，所以不需要使用到`import`语句。

运行`swift build`并尝试`Hello`的新版本：

```
$ swift build
$ .build/debug/Hello 'whoami'
```

目前为止，你已经能够运用开源Swift来运行一些你想要的程序了。接下来我们就可以进入正题开始入手SPM.

### 快速入门实例

在本章节中，我们简单地学会了编译一个"`Hello world"程序。

为了了解SPM究竟能做什么，我们来看一下下面这个由4个独立的包组成的例子:

* [O2PlayingCard](https://github.com/marklin2012/O2PlayingCard.git)-定义了O2PlayingCard ， O2Suit ， O2Rank ， 3个类型
* [O2FisherYates](https://github.com/marklin2012/O2FisherYates.git)-定义了 shuffle() 和 shuffleInPlace() 方法实现的扩展
* [O2DeckOfPlayingCards](https://github.com/marklin2012/O2DeckOfPlayingCards.git)-定义了一个 O2Deck 类型对 O2PlayingCard 值得数据进行洗牌和抽牌。
* [O2Dealer](https://github.com/marklin2012/O2Dealer.git)-定义了一个用来创建 O2DeckOfPlayingCards 进行洗牌和抽出前10个卡片的可执行文件。

你可以从[O2Dealer from GitHub ](https://github.com/marklin2012/O2Dealer.git)编译并运行完整例子，然后运行如下命令:

```
$ cd O2Dealer
$ swift build
$ .build/debug/O2Dealer
```

### 创建一个库包

我们将从创建一个代表一副标准的52张扑克牌的模块开始。 O2PlayingCard 模块定义了 由 O2Suit 枚举值（Clubs, Diamonds, Hearts, spades）和 O2Rank 枚举值（Ace, Two, Three, …, Jack, Queen, King）组成的 O2PlayingCard 类。各个类的核心代码如下：

```
public enum O2Rank : Int {
    case Ace = 1
    case Two, Three, Four, Five, Six, Seven, Eight, Nine, Ten
    case Jack, Queen, King
}

public enum O2Suit: String {
    case Spades, Hearts, Diamonds, Clubs
}

public struct O2PlayingCard {
    let rank: O2Rank
    let suit: O2Suit
}
```

一般来说，一个包包括位于Source/的源文件

```
O2PlayingCard
├── Sources
│   ├── O2PlayingCard.swift
│   ├── O2Rank.swift
│   └── O2Suit.swift
└── Package.swift
```

由于`O2PlayingCard `模块并不会生成可执行文件，这里应该成为库。库表示被编译成一个可以被其它包导入的模块的包，默认情况下，库模块公开所有位于`Sources/`目录下的源代码中声明的公共类型的方法。


运行 swift build 开始启动 Swift 编译的过程。如果一切进行顺利，将会在 .build/debug 目录下生成 O2PlayingCard.build 目录。

接下来，我们在`Package.swift`文件中定义包名，代码如下：

```
import PackageDescription

let package = Package(
  name: "O2PlayingCard"
)
```

然后我们只要将`O2PlayingCard `提交到Github上，并且给他发布一个Release版本即可完成该库包，这里可以自己手动添加一个`.gitignore`文件，忽略掉`/.build`，因为我们的包是不需要包括生成的编译结果的内容的。


## 使用编译配置语句

下一个即将编译的模块是`O2FisherYates `.跟之前`O2PlayingCard `有所不同，该模块没有定义新的类，取而代之的是该模块拓展了一个已经存在的特殊的`CollectionType `和`MutableCollectionType `接口协议，用来添加`shuffle()`方法和对应的`shuffleInPlace() `方法。

在 OS X 中，系统模块是 Darwin , 提供的函数是 arc4random_uniform(_:) 。在 Linux 中， 系统模块是 Glibc ， 提供的函数是 random() ：

```
#if os(Linux)
  import Glibc
#else
  import Darwin.C
#endif

public extension Collection {
  func shuffle() -> [Generator.Element] {
    var array = Array(self)
    array.shuffleInPlace()
    
    return array
  }
}

public extension MutableCollection where Index == Int {
  mutating func shuffleInPlace() {
    guard count > 1 else { return }
    v 
    for i in 0..<count - 1 {
      #if os(Linux)
        let j = Int(random() % (count - i)) + i
      #else
        let j = Int(arc4random_uniform(UInt32(count - i))) + i
      #endif
      guard i != j else { continue }
      swap(&self[i], &self[j])
    }
  }
}
```

剩下的步骤和前面的类似，编译通过后上传到GitHub,发布Release版本。

### 导入依赖
`O2DeckOfPlayingCards `包把前两个包聚合到一起：它定义了一个`O2PlayingCard `数组中使用`O2FisherYates `的`shuffle()`方法的Deck类型。

为了使用 O2FisherYates 和 O2PlayingCards 模块， O2DeckOfPlayingCards 包必须在 Package.Swift 清单中将上述模块声明为依赖。

```
import PackageDescription

let package = Package(
    name: "O2DeckOfPlayingCards",
    dependencies: [
        .Package(url: "https://github.com/marklin2012/O2PlayingCard.git",
                 majorVersion: 1),
        .Package(url: "https://github.com/marklin2012/O2FisherYates.git",
                 majorVersion: 1),
    ]
)
```
每个依赖都需要指定一个源URL和版本号，源URL是指允许当前用户解析到对应的Git仓库。版本号遵循 [语义化版本号 2.0.0](http://semver.org/lang/zh-CN/) 的约定,用来决定检出或者使用哪个Git标签版本来建立依赖。对于`FisherYates `和`PlayingCard `这两个依赖来说， 最新的将要被使用的主版本号为1.

当你运行`swift build`命令时，包管理器将会下载所有的依赖，并将它们编译成静态库，再把它们链接到包模块中。这样将会使`O2DeckOfPlayingCards `可以访问依赖import语句的模块的公共成员

你可以看到这些资源被下载到你工程根目录的 Packages 目录下，并且会生成编译产品在你工程根目录的 .build 目录下。

```
O2DeckOfPlayingcards
├── .build
│   └── debug
│       ├── O2DeckOfPlayingCards.build
│       ├── O2DeckOfPlayingCards.swiftdoc
│       ├── O2DeckOfPlayingCards.swiftmodule
│       ├── O2FisherYates.build
│       ├── O2FisherYates.swiftdoc
│       ├── O2FisherYates.swiftmodule
│       ├── O2PlayingCard.build
│       ├── O2PlayingCard.swiftdoc
│       └── O2PlayingCard.swiftmodule
└── Packages
    └── O2FisherYates-1.0.0
    │   ├── Package.swift
    │   ├── README.md
    │   └── Sources
    └── O2Playingcard-1.0.1
        ├── Package.swift
        ├── README.md
        └── Sources
```


`Package`目录包含了被复制的包依赖的所有仓库，这样将使你能修改源代码并直接推送这些修改到它们的源，而不需要再对每个包在单独进行复制。


Swift是一门先进的语言，SPM的社区也在不断地完善中。在swift开源之后，我们很容可以看到它的潜力，看来掌握这门语言必将是一个大趋势。


