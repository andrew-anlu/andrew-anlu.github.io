---
layout: post
title: "Nodejs基础"
date: 2016-05-21 12:45:39 +0800
comments: true
categories: NodeJs
---

* [什么是NodeJS](#什么是NodeJS)
* [用途](#用途)
* [安装](#安装)
* [运行](#运行)
* [模块](#模块)
* [小结](#小结)

<!--more-->

## <a name="什么是NodeJS">什么是NodeJS</a>
JS是脚本语言，脚本语言都需要一个解析器才能运行。对于写在 HTML 页面里的 JS，浏览器充当了解析器的角色。而对于需要独立运行的 JS，NodeJS 就是一个解析器。

每一种解析器都是一个运行环境，不但允许JS定义各种数据结构，进行各种计算，还允许 JS 使用运行环境提供的内置对象和方法做一些事情。例如运行在浏览器中的JS的用途是操作 DOM，浏览器就提供了 document 之类的内置对象。而运行在 NodeJS 中的 JS 的用途是操作磁盘文件或搭建 HTTP 服务器，NodeJS 就相应提供了 fs、http 等内置对象。



## <a name="用途">用途</a>

尽管存在一听说可以直接运行JS文件就觉得很酷的同学，但大多数同学在接触新东西时首先关心的有啥用户，以及能带来什么价值

NodeJS的作者说，他创造NodeJS的目的是为了实现提高性能web服务器，他首先看重的是事件机制和异步IO模型的优越性，而不是JS。但是他需要选择一种编程语言实现他的想法，这种编程语言不能自带IO功能，并且需要能良好支持事件机制。JS没有自带IO功能，天生就用于处理浏览器中的DOM事件，并且拥有一大群程序员，因此就成了天然的选择。

如他所愿，NodeJs在服务器端活跃起来，出现了大批基于NodeJs的web服务。而另一方面，NOdeJS让前端如获神器，终于可以让自己的能力覆盖范围跳出浏览器窗口，更大批的前端工具如雨后春笋。

因此，对于前端而言，虽然不是人人都要拿NodeJs写一个服务器端程序，但简单使用命令交互模式调试JS片段，复杂可至编写工具提升工作效率。

NodeJS生态圈正欣欣向荣





## <a name="安装">安装</a>
NodeJs提供了一些安装程序，都可以在nodejs.org这里下载并安装

在MacOs系统下，选择.pkg后缀的安装文件.

### 编译安装
Linux系统下没有现成的安装程序可用，虽然一些发行版可以使用 apt-get之类的方式安装，但不一定能安装到最新版。因此Linux系统下一班使用以下方式编译方式安装NodeJs。

1. 确保系统下 g++版本在4.6以上，python版本在2.6以上
2. 从nodejs.org下载tar.gz后缀的NodeJS最新版源代码包并解压到某个位置
3. 进入解压到的目录，使用以下命令编译和安装

```
$ ./configure

$ make

$ sudo make install
```

## <a name="运行">运行</a>
打开终端，键入node进入命令交互模式，可以输入一条语句后立即执行并显示效果，例如:

```
$ node
> console.log('Hello World!');
Hello World!
```

如果要运行一大段代码的话，可以先写一个JS文件再运行，例如有以下hello.js

```
function hello() {
    console.log('Hello World!');
}
hello();
```

写好后，在终端键入 node hello.js运行，结果如下:

```
$ node hello.js
Hello World!
```

### 权限问题
在Linux系统下，使用NodeJs监听80或者443端口提供HTTP(s)服务时需要root权限，有两种方式可以做到.

一种方式是使用sudo命令运行NodeJs,例如通过以下命令运行的server.js中有权限使用80和443端口，一般推荐这种方式，可以保证仅为有需要的js脚本提供root权限。


```
$ sudo node server.js
```
另一种方式是使用chmod + s 命令让NodeJs总是以root权限运行，具体做法如下，因为这种方式让任何JS脚本都有了root权限，不太安全，因此在需要很考虑安全下不推荐使用：


```
$ sudo chown root /usr/local/bin/node
$ sudo chmod +s /usr/local/bin/node
```


## <a name="模块">模块</a>

编写稍大一点的程序时一般都会讲代码模块化，在NOdeJs中，一般将代码合理拆分到不同的JS文件中，每一个文件就是一个模块，而文件路径就是模块名.

在编写每一个模块时，都有 `require`,`exports`,`module`三个预先定义好的变量可供使用。


###3 Require
require函数用于在当前模块中加载和使用别的模块，传入一个模块名，返回一个模块导出对象。模块名可使用相对路径(以./开头)，或者是绝对路径(以/或C:之类的盘符开头).另外，模块名中的.js扩展名可以省略。以下是一个例子:

```
var foo1 = require('./foo');
var foo2 = require('./foo.js')
var foo3 = require('/home/user/foo')
var foo4 = require('/home/user/foo.js')
```

//foo1至foo4中保存的是统一模块的导出对象。另外可以使用以下方式加载和使用一个JSON文件。

```
var data = require('./data.json')
```

### exports
exports对象是当前模块的导出对象，用于导出模块共有方法和属性。别的模块通过require函数使用当前模块时得到的就是当前模块的exports对象。以下例子中导出了一个共有方法.

```
exports.hello = function () {
    console.log('Hello World!');
};
```
以上代码中，模块默认导出对象被替换为一个函数。

### 模块初始化
一个模块中的JS代码仅在模块第一次被使用时执行一次，并在执行过程中初始化模块的导出对象。之后，缓存起来的导出对象被重复利用。

### 主模块
通过命令行参数传递给NodeJS以启动程序的模块被称为主模块。住模块负责调度祖成整个程序的其它模块完成工作，例如通过以下命令启动程序时，main.js就是主模块.

```
$ node main.js
```

完整示例:

例如有以下目录:

```
- /home/user/hello/
    - util/
        counter.js
    main.js
```

其中counter.js内容如下:

```
var i = 0;

function count() {
    return ++i;
}

exports.count = count;
```

该模块内部定义了一个私有变量i,并在exports对象导出了一个共有方法count.

主模块main.js内容如下：

```
var counter1 = require('./util/counter');
var    counter2 = require('./util/counter');

console.log(counter1.count());
console.log(counter2.count());
console.log(counter2.count());
```
运行该程序的结果如下:

```
$ node main.js
1
2
3
```

可以看到，counter.js并没有因为被require了两次而初始化两次

## <a name="小结">小结</a>

本章介绍了有关NodeJs的基本概念和使用方法，总结起来有以下知识点:

* NodeJs是一个js脚本解析器，任何操作系统下安装NodeJS本质上做的事情都是把nodejs执行程序复制到一个目录，然后保证这个目录在系统PATH环境变量下，以便终端下可以使用node命令
* 终端下直接输入node命令可进入命令交互模式，很适合用来测试一些JS代码片段，比如正则表达式
* nodejs使用CMD模块系统，主模块作为程序入口点，所有模块在执行剁成中只初始化一次。
* 除非JS模块不能满足需求，否则不要轻易使用二进制模块，否则你的用户会叫苦连天
* 




