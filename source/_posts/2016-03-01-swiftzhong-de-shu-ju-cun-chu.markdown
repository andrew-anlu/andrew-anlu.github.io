---
layout: post
title: "swift中的数据存储"
date: 2016-03-01 10:52:34 +0800
comments: true
categories: Swift
---
程序就是对数据进行处理的，存储数据在app端也是很有必要的。本章将学习如何使用文件系统在磁盘上存储信息，如何在内置的用户偏好设置数据库中存储简单数据。
<!--more-->

##NSuserDefaults

NSUserDefaults类允许以"键值"形式保存设置信息。我们不需要加载和读取设置文件，偏好设置会被自动保存。

要访问存储在NSuserDefaults中的偏好设置，需要NSuserDefaults类的一个实例。为得到这样的一个实例，必须使用NSUserdefaults类额standardUserDefaults:

	let defaults=NSUserDefaults.standardUserDefaults()

要在默认对象中注册默认值，首先需要创建一个字典。这个字典的键与偏好设置的名字相同，与这些键关联的值就是这些设置的默认值。一旦有个这个字典，就用registerDefaults方法将它提供给默认对象

	let meDefaults=["greeting":"hello",
                "numberOfItems":1]
    defaults.registerDefaults(meDefaults)
    
完成这一工作后，就可以使用默认对象的值了

用reigisterDefaults方法注册的"默认设置"并不是村村在磁盘上，也就是说在应用程序每次启动时都需要调用它。但在应用程序中设定的“默认设置”会被保存。

有了对一个NSuserdefaults对象的引用，就可以向对待字典一样对待NSUserdaults对象，我们可以使用ObjectForkey方法从默认对象中提取一个值

	let greeting=defaults.objectForKey("greeting") as? String
	
只有少数对象可以村村在默认对象中。可以存储在默认对象中的对象只有属性列表对象。

 * 字符串
 * 数字
 * NSData
 * NSdate
 * 数组和字典

 如果需要在默认对象中存储任何其他类型的对象，应当首先对其存档(archive),转换为NSData
 
 
除了从默认对象中提取值之外，还可以设定值。在NSUserDefaults对象中设定值时，这个值将被永远保存
要在NSUserDefaults对象中设定一个对象，可以使用SetObject(_,forKey)

	let newGreeting="hi ,there";
    defaults.setObject(newGreeting, forKey: "greeting")  
    
##使用NSFileManager
应用程序与文件系统的界面是NSFileManager对象，利用它可以列出文件夹的内容，创建和重命名和删除文件，要访问NSFilemanager类，可以使用共享管理器对象

	let fileManager=NSFileManager.defaultManager()

###注意
NSFileManager允许我们对其设定一个委托，在文件管理器完成诸如复制或移动文件之类的操作时，它会受到消息。在使用这一功能时，应当创建你自己的NSFileManager实例，而不是使用共享对象

	let fileManager=NSFileManager()
	fileManager.delegate=self;



* 获取一个临时目录

    //获取临时目录
    let tempraryDirectoryPath=NSTemporaryDirectory()
这个函数会返回一个字符串，它包含了可以在其中存储文件的目录路径。如果希望以NSURL形式使用它，就需要用fileURLWithPath方法进行转换。

* 创建目录

利用NSFileManager，可以在文件系统上创建和删除项目。比如，要创建一个新目录，可以利用:

* 创建文件
创建文件的过程也是一样的，我们在NSString中提供一个路径，文件应当包含的NSData,以及一个可选的字典，其中列出此文件应当拥有的属性：

```
fileManager.createFileAtPath(newFilePath!,
      contents:NewFileData,
      attributes:nil)
```
      
* 删除文件

```
fileManager.reomveItemAtURL(newFileURL!,error:&error)
```

		  
* 移动和复制文件

```
fileManager.moveItemAtURL(sourceURL!,toURL:destinationURL,error:nil)
```

        
要复制一个项目，可以这样做

```
fileManager.copyItemAtURL(sourceURL!,toURL:destinationURL,error:nil)
```

	
      
	
