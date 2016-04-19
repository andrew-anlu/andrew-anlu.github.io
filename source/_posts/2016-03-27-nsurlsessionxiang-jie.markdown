---
layout: post
title: "NSUrlSession详解"
date: 2016-03-27 18:48:28 +0800
comments: true
categories: iOS
---

NSUrlSession是NSUrlConnection的替代品。

NSUrlConennection指的是一组构成 Foundation框架中URL加载系统的相互关联的组件:NSURLRequest,NSUrlResponse,NSURlProtocol等,在协商发送一个请求到服务器的过程中，该服务器可发出验证质询，这可以由共享的cookie，证书存储（credential storage）或通过连接委托自动处理。必要的时候，为了无缝地改变装载行为，传出请求也可以被注册的NSURLProtocol对象截获.

<!--more-->

不管怎样，考虑到NSURLConnection作为一个网络基础架构，成千上万的Cocoa和Cocoa Touch应用程序从中获益，它已经表现得相当好。但是，这些年来，iPhone和iPad新兴的用例，特别是有一些已经向NSURLConnection的几个核心设想提出了挑战，对其重构已经迫在眉睫。

在2013年的WWDC上，Apple揭开了NSURLConnection继任者的面纱：NSURLSession.

与NSUrlConnection类似，除了同名类 NSUrlsession,NSUrlSession指的是一组相互依赖的类，NSURlSession包括与之前相同的组件，例如NSUrlRequest,NSURLCatch等等。
　　
##NSURlconenction 与 NSSession的不同
　　与NSUrlConnection相比，NSUrlSession最直接的改善就是提供了配置每个回话的缓存，协议，cookie和证书策略(credential policies),甚至跨应用程序共享它们的能力。这使得框架的网络基础架构和部分应用程序独立工作，而不会相互干扰，每一个NSUrlSession对象都是根据一个NSURlSessionConfiguration初始化的，
　　
　　

该NSURlSessionConfiguration指定了上面提到的策略，一级一系列为了提高移动设备性能而专门添加的新选项。

NSUrlSession的另一个重要组成部分就是会话任务，它负责处理数据的加载，以及客户端与服务器之间的文件和数据的上传和下载。


##NSURLSession简介

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/Screen-Shot-2015-08-20-at-12.27.21-am.png)


NSURLsession关键对象负责接收和发送http请求，创建NSURlSessionConfiguration,这里有三种方式:

* `defaultSessionConfiguration `创建一个默认的配置文件，用户可以存储缓存，创建证书和缓存cookie等
* `ephemeralSessionConfiguration `和默认配置文件很相似，除了它可以在内存中存储之外，它更像是一个私有的session
* `backgroundSessionConfiguration `这个配置文件支持上传和下载任务在后台。当程序挂起或者被终止之后任务可以继续执行。

`NSURLSessionConfiguration `依然可以让你配置session的属性，比如设置超时时间，缓存策略和http请求头等。[这里](https://developer.apple.com/library/mac/documentation/Foundation/Reference/NSURLSessionConfiguration_class/index.html#//apple_ref/occ/cl/NSURLSessionConfiguration)有完整的配置文档。

`NSURLSessionTask `是一个抽象的任务符号,一个session创建一个任务，它不仅可以请求数据，还可以上传和下载。

这里有三种类型的任务：

* `NSURLSessionDataTask `：用这个任务可以发送http请求，从而从服务器得到返回的数据
* `NSURLSessionUploadTask `：用这个任务可以从本地硬盘上往服务器上传文件，一般是HTTP post请求或者PUT请求.
* `NSURLSessionDownloadTask `:用这个任务可以从远程服务器上下载文件

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/Screen-Shot-2015-08-20-at-12.27.27-am.png)

你可以挂起，回滚和取消任务，`NSURLSessionDownloadTask `还有一个特性就是支持断点下载。

一般来讲, NSURLSession 有两种方式返回数据：

1. 利用completion handler,当任务完成之后，不管是成功返回还是产生错误；
2. 还有一种就是利用NSSession的代理，依然可以捕获到返回的数据;

##编写实例Demo
[启动工程在这里下载](http://www.raywenderlich.com/wp-content/uploads/2016/01/HalfTunes-Starter.zip),

开始做一个 在Itunes搜索歌曲，通过[Itunes API](https://www.apple.com/itunes/affiliates/resources/documentation/itunes-store-web-service-search-api.html)下载歌曲的这么个小工程，支持暂停，下载功能。

下完工程，运行效果如下:
![itunes](http://7xkxhx.com1.z0.glb.clouddn.com/Simulator-Screen-Shot-12-Aug-2015-11.10.57-pm-281x500.png)


###开始编写代码
你可以添加代码去查询itunes中的歌曲，通过查找 Itunes search Api.

在`SearchViewController.swift`文件中，添加如下代码:

```
// 1
let defaultSession = NSURLSession(configuration: NSURLSessionConfiguration.defaultSessionConfiguration())
// 2
var dataTask: NSURLSessionDataTask?
```
上面的代码做了如下工作:

1. 用默认的配置文件创建NSURLSession
2. 你定义了一个`NSURLSessionDataTask `变量，用它发送http请求，这个任务将会被重复初始化和重复利用在用户创建一个新查询的时候

现在，替换`searchBarSearchButtonClicked(_:)`里面的代码:

```
func searchBarSearchButtonClicked(searchBar: UISearchBar) {
  dismissKeyboard()
 
  if !searchBar.text!.isEmpty {
    // 1
    if dataTask != nil {
      dataTask?.cancel()
    }
    // 2
    UIApplication.sharedApplication().networkActivityIndicatorVisible = true
    // 3
    let expectedCharSet = NSCharacterSet.URLQueryAllowedCharacterSet()
    let searchTerm = searchBar.text!.stringByAddingPercentEncodingWithAllowedCharacters(expectedCharSet)!
    // 4
    let url = NSURL(string: "https://itunes.apple.com/search?media=music&entity=song&term=\(searchTerm)")
    // 5
    dataTask = defaultSession.dataTaskWithURL(url!) {
      data, response, error in
      // 6
      dispatch_async(dispatch_get_main_queue()) {
        UIApplication.sharedApplication().networkActivityIndicatorVisible = false
      }
      // 7
      if let error = error {
        print(error.localizedDescription)
      } else if let httpResponse = response as? NSHTTPURLResponse {
        if httpResponse.statusCode == 200 {
          self.updateSearchResults(data)
        }
      }
    }
    // 8
    dataTask?.resume()
  }
}
```

运行后代码如下:
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160401-1.png)

如果出现错误`An SSL error has occurred and a secure connection to the server cannot be made.`

请在info.plist中配置，在Info.plist中添加NSAppTransportSecurity类型Dictionary。 在NSAppTransportSecurity下添加NSAllowsArbitraryLoads类型Boolean,值设为 YES

上面的代码步骤意义如下:

1. 检查用户每一次查询，dataTask是否已经初始化，如果没有，则取消该任务
2. 设置的状态栏上的转子运行起来，证明数据正在请求当中
3. 当用户输入查询参数之前，调用 请求字符串的`stringByAddingPercentEncodingWithAllowedCharacters(_:)`,确保是一个正确的URL.　(这个 text 的类型是 String ，常用于搜索功能，在  URL 中包含被搜的关键字，如果不处理搜中文或者带空格的英文会直接崩溃);
4. 下一步创建一个NSURL用上面的（安全的）字符串，使用GET请求去调用Itunes Search API
5. 从创建的Session中，你初始化`NSURLSessionDataTask `去处理http请求，这个`NSURLSessionDataTask `任务使用completion handler （回调函数）去响应服务器返回的数据
6. 异步调用主线程，在主线程上隐藏网络请求的转子
7. 如果http请求是成功的，你可以调用`updateSearchResults(_:)`来刷新表格数据，返回数据是NSData类型的，需要在updateSearchResults方法中进行处理
8. 所有任务默认是挂起状态，需要你调用 `resume() `去启动任务


##下载


看着搜索到的歌曲，感觉页面不错，但是如果我们能够通过点击 download,然后把歌曲下载到本地是不是更爽呢?下一步让我实现这个功能点.

用多线程实现下载是容易的。首先你要创建一个新的文件命名为 `Download.swift`. 打开这个文件，添加如下代码:

```
class Download: NSObject {
 
  var url: String
  var isDownloading = false
  var progress: Float = 0.0
 
  var downloadTask: NSURLSessionDownloadTask?
  var resumeData: NSData?
 
  init(url: String) {
    self.url = url
  }
}
```

属性说明如下:

* URL:下载文件的url地址，它也扮演着唯一的标识符在下载过程中
* isDownloading:是否正在下载或暂停
* progress : 下载的进度,[0-1]
* downloadTask: NSURLSessionDownloadTask下载任务
* resumeData:当你暂停一个下载任务时，它负责存储此时的数据量；如果后台服务器支持的话，当用户再次点击继续下载，它会从这里开始继续下载这个文件，俗称 断点下载


切换到 `SearchViewController.swift`，添加如下代码:

```
var activeDownloads = [String: Download]()
```
##创建下载任务
准备工作做得差不多了，现在你只需实现下载，首先你要创建一个session去实现下载任务.

在 `SearchViewController.swift`文件中，在`viewDidLoad():`之前添加如下代码:

```
lazy var downloadsSession: NSURLSession = {
  let configuration = NSURLSessionConfiguration.defaultSessionConfiguration()
  let session = NSURLSession(configuration: configuration, delegate: self, delegateQueue: nil)
  return session
}()
```
这里初始化了一个session,用默认的配置文件，去处理所有的下载任务，你也可以指定delegate,这将会使你收到 `NSURLSession `的代理调用，这个是很有用的，它能有效的跟踪下载任务下载的进度和是否下载完成等。

设置代理的队列是nil,会促使session创建一个操作队列，默认的去调用代理方法和回调方法.

在创建`downloadsSession `属性的时候，我们加了`lazy`特性，这会让你延迟加载这个session直到你需要它的时候，更重要的是，它会通过`self`作为代理参数去初始化，假如`self`还没有初始化。

在`SearchViewController.swift `文件中，找到空的`NSURLSessionDownloadDelegate `并且扩展如下:

```
extension SearchViewController: NSURLSessionDownloadDelegate {
  func URLSession(session: NSURLSession, downloadTask: NSURLSessionDownloadTask, didFinishDownloadingToURL location: NSURL) {
    print("Finished downloading.")
  }
}
```

`NSURLSessionDownloadDelegate `定义了代理方法你需要去实现，在你使用 NSURLSession 下载任务的时候，这个唯一的不是可选的代理方法是 ` URLSession(_:downloadTask:didFinishDownloadingToURL:),`,当下载完成的时候，将会执行这个代理方法，打印简答的一句话.

在`SearchViewController.swift`文件中，替换`startDownload(_:) `这个方法的代码如下:

```
func startDownload(track: Track) {
  if let urlString = track.previewUrl, url =  NSURL(string: urlString) {
    // 1
    let download = Download(url: urlString)
    // 2
    download.downloadTask = downloadsSession.downloadTaskWithURL(url)
    // 3
    download.downloadTask!.resume()
    // 4
    download.isDownloading = true
    // 5
    activeDownloads[download.url] = download
  }
}
```

当你点击 `Download`按钮的时候，你将会调用` startDownload(_:) `函数去执行下载命令，上面的代码执行的步骤如下:

1. 你用URL去初始化一个DownLoad对象
2. 使用上面的NSURL和downloadsSession去初始化`NSURLSessionDownloadTask `
3. 调用resume()去启动一个下载任务
4. 设置下载标识 为true
5. 最后，你把下载的URL作为key,download对象作为值放到一个字典中


编译运行你的项目，查询出来的歌曲中，点击 Download按钮，你将会看到一个消息打印在控制台。
`Finished downloading.`

##保存&播放
当下载任务完成的时候，`URLSession(_:downloadTask:didFinishDownloadingToURL:)`提供了一个URL存储临时文件路径，你的工作就是移动它到你程序的沙盒当中，在这个方法返回之前。当然，这个过程当中你需要删除全局字典中正在下载的download对象，并且更新表格.

你需要添加一个Helper方法简化这个操作，在` SearchViewController.swift`中，添加如下方法:

```
func trackIndexForDownloadTask(downloadTask: NSURLSessionDownloadTask) -> Int? {
  if let url = downloadTask.originalRequest?.URL?.absoluteString {
    for (index, track) in searchResults.enumerate() {
      if url == track.previewUrl! {
        return index
      }
    }
  }
  return nil
}
```
这个方法仅仅返回 当前的URL在 searchResults集合中的索引，下一步，替换 URLSession(_:downloadTask:didFinishDownloadingToURL:) 中的代码:

```
func URLSession(session: NSURLSession, downloadTask: NSURLSessionDownloadTask, didFinishDownloadingToURL location: NSURL) {
  // 1
  if let originalURL = downloadTask.originalRequest?.URL?.absoluteString,
    destinationURL = localFilePathForUrl(originalURL) {
 
    print(destinationURL)
 
    // 2
    let fileManager = NSFileManager.defaultManager()
    do {
      try fileManager.removeItemAtURL(destinationURL)
    } catch {
      // Non-fatal: file probably doesn't exist
    }
    do {
      try fileManager.copyItemAtURL(location, toURL: destinationURL)
    } catch let error as NSError {
      print("Could not copy file to disk: \(error.localizedDescription)")
    }
  }
 
  // 3
  if let url = downloadTask.originalRequest?.URL?.absoluteString {
    activeDownloads[url] = nil
    // 4
    if let trackIndex = trackIndexForDownloadTask(downloadTask) {
      dispatch_async(dispatch_get_main_queue(), {
        self.tableView.reloadRowsAtIndexPaths([NSIndexPath(forRow: trackIndex, inSection: 0)], withRowAnimation: .None)
      })
    }
  }
}
```

上面的代码做的事情如下:

1. 定义了两个变量，`originalURL`请求路径的url,`destinationURL `变量则是通过`localFilePathForUrl(_:)`方法生成的，该方法会获取当前程序的沙盒路径再追加 传递的URL的最后一个后缀(/)的路径作为返回的参数。作为目标文件夹的路径
2. 使用NSFileManager,在开始拷贝文件之前，先删除目标文件夹下的文件，如果存在的话。然后进行拷贝从本地拷贝到目标文件夹
3. 删除download从全局的download字典中
4. 最后刷新表格对应的哪一行

编译运行你的工程，点击搜索然后选中一行进行下载，当下载完成时候，在控制台会打印一行信息,下载的目标路径

```
file:///Users/Andrew/Library/Developer/CoreSimulator/Devices/875165C2-FA55-4884-96AE-A7C8E3223C12/data/Containers/Data/Application/52B47648-04A2-4C26-8BCF-F41D2C76CA21/Documents/mzm.gyadmzom.aac.p.m4a
```

这时下载按钮将会消失，再次点击表格的对应的那行，将会弹出一个 MPMoviePlayerViewController,开始播放音频.

##监视下载进度
当然，现在你还没有监视下载的进度条，为了提高用户体验，你将要改变你的App去监听下载的进度在每个cell中。

在SearchViewController.swift文件中,找到 `NSURLSessionDownloadDelegate`的扩展，然后添加如下的代理方法:

```
func URLSession(session: NSURLSession, downloadTask: NSURLSessionDownloadTask, didWriteData bytesWritten: Int64, totalBytesWritten: Int64, totalBytesExpectedToWrite: Int64) {
 
    // 1
    if let downloadUrl = downloadTask.originalRequest?.URL?.absoluteString,
      download = activeDownloads[downloadUrl] {
      // 2
      download.progress = Float(totalBytesWritten)/Float(totalBytesExpectedToWrite)
      // 3
      let totalSize = NSByteCountFormatter.stringFromByteCount(totalBytesExpectedToWrite, countStyle: NSByteCountFormatterCountStyle.Binary)
      // 4
      if let trackIndex = trackIndexForDownloadTask(downloadTask), let trackCell = tableView.cellForRowAtIndexPath(NSIndexPath(forRow: trackIndex, inSection: 0)) as? TrackCell {
        dispatch_async(dispatch_get_main_queue(), {
          trackCell.progressView.progress = download.progress
          trackCell.progressLabel.text =  String(format: "%.1f%% of %@",  download.progress * 100, totalSize)
        })
    }
  }
}
```
通过这个代理方法做的工作如下:

1. 使用提供的 downloadTask 找到URL属性，然后从全局激活的下载字典中查找 DownLoad对象
2. 这个方法将会返回总的字节数和已经写入的字节数，你可以利用这个两个值算出当前的下载进度并且实时更新进度条。
3. NSByteCountFormatter 将会把字节数转化成人类能够看懂的文件大小，有将会使用这个字符串去显示下载的文件大小和百分比
4. 最后，你将要定位到这个Cell,在主线程中更新进度条和显示的百分比

下一步，你将要配置这个cell属性去显示进度条

找到下面的代码在 `tableView(_:cellForRowAtIndexPath:):`中

```
let downloaded = localFileExistsForTrack(track)
```
添加如下代码在这行的上面:

```
var showDownloadControls = false
if let download = activeDownloads[track.previewUrl!] {
  showDownloadControls = true
 
  cell.progressView.progress = download.progress
  cell.progressLabel.text = (download.isDownloading) ? "Downloading..." : "Paused"
}
cell.progressView.hidden = !showDownloadControls
cell.progressLabel.hidden = !showDownloadControls
```

为了跟踪正在下载的歌曲，你将要设置 showDownloadControls为true,否则，你将要设置为false.你将要显示这进度条和文字。

为了暂停任务，显示"Paused"状态，否则，显示 “Downloading....”
最后，替换这行代码:

```
cell.downloadButton.hidden = downloaded
```

使用下面这行代码:

```
cell.downloadButton.hidden = downloaded || showDownloadControls
```

到这，你可以告诉表格是否隐藏下载按钮。

编译运行你的工程，点击下载按钮，你将要看到一个进度条和下载的进度，以及下载的百分比。
![logo](http://7xkxhx.com1.z0.glb.clouddn.com/Screen-Shot-2015-08-17-at-11.02.03-pm-480x78.png)

OK,你做到了!😀


##暂停 恢复  取消下载任务

假如我需要暂停一个任务，或者取消任务？此时该怎么做呢？

在这个章节，你将要实现暂停，恢复，取消任务操作。

你将要开始编写代码，通过允许用户去取消一个正在激活的任务
替换 `cancelDownload(_:)` 使用下面的代码:

```
func cancelDownload(track: Track) {
  if let urlString = track.previewUrl,
    download = activeDownloads[urlString] {
      download.downloadTask?.cancel()
      activeDownloads[urlString] = nil
  }
}
```

为了取消任务，你可以从全局激活的字典中取出Download对象，然后调用它的 cancel() 方法，执行取消命令，然后在全局字典中移除它

暂停任务和取消任务非常类似，不同点在于当你暂停一个任务的时候，会产生恢复数据，它包含了足够多的信息去恢复下载数据，当然后台服务器必须要支持这个特性才能完全实现断点下载功能.

>**注意**
>你能恢复一个下载任务在一定的控制条件下，例如，从你第一下请求下载开始，这个下载资源就不能再改变了。想要更详细的控制条件，请参考苹果的 [官方文档](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSessionDownloadTask_class/index.html#//apple_ref/occ/instm/NSURLSessionDownloadTask/cancelByProducingResumeData:)
>
>


现在，替换 `pauseDownload(_:) `使用下面的代码:


```
func pauseDownload(track: Track) {
  if let urlString = track.previewUrl,
    download = activeDownloads[urlString] {
      if(download.isDownloading) {
        download.downloadTask?.cancelByProducingResumeData { data in
          if data != nil {
            download.resumeData = data
          }
        }
        download.isDownloading = false
      }
  }
}
```

这个关键字是不同的 cancelByProducingResumeData(_:) 替代了 cancel(),你检索这个恢复的数据从这个方法cancelByProducingResumeData提供的回调函数中，并且把恢复的数据保存到Download的resumeData属性中。并且设置isDownloading=false


下面替换`resumeDownload(_:)`用下面的代码:

```
func resumeDownload(track: Track) {
  if let urlString = track.previewUrl,
    download = activeDownloads[urlString] {
      if let resumeData = download.resumeData {
        download.downloadTask = downloadsSession.downloadTaskWithResumeData(resumeData)
        download.downloadTask!.resume()
        download.isDownloading = true
      } else if let url = NSURL(string: download.url) {
        download.downloadTask = downloadsSession.downloadTaskWithURL(url)
        download.downloadTask!.resume()
        download.isDownloading = true
      }
  }
}
```

当用户恢复一个下载任务时，你检查下当前的Download对象 恢复数据的属性是否有值，如果有值，你将会创建一个新的下载任务通过 `downloadTaskWithResumeData(_:)`和一个恢复数据的参数，启动`resume()`恢复数据的命令;如果这个 恢复数据的属性是空的或者其他一些原因，你将要用URL创建一个新的下载任务，启动它.

在上面的案例中，你设置这个isDownloading为true,表明任务正在进行.

还有一件事件要做就是设置这三个函数的工作属性，你需要在cell中显示 或者隐藏 `暂停`，`取消`和`继续下载`按钮。

找到 `tableView(_:cellForRowAtIndexPath:)`然后找到下面这行代码:

```
if let download = activeDownloads[track.previewUrl!] {
```
然后添加下面的代码:

```
let title = (download.isDownloading) ? "Pause" : "Resume"
cell.pauseButton.setTitle(title, forState: UIControlState.Normal)
```

暂停和继续下载按钮共用一个按钮。

下一步，添加下面的代码在 `tableView(_:cellForRowAtIndexPath:)`结尾:

```
cell.pauseButton.hidden = !showDownloadControls
cell.cancelButton.hidden = !showDownloadControls
```

当下载任务激活的时候，这里仅仅把按钮显示出来。

编译运行你的工程，下载几个歌曲试试，你可以暂停，继续下载，取消 下载任务。

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/Simulator-Screen-Shot-18-Aug-2015-10.14.38-pm-281x500.png)

##支持后台下载
你的App现在看来已经很不错了，但是还有一个问题，是否支持后台任务下载:当你的程序进入后台模式或者因为别的原因意外终止了，后台任务是否能继续下载?

假如你的App不再运行了，那它怎么能继续工作呢？这儿有一个守护进程在App运行之外，去管理后台任务下载；它发送一个适当的代理方法通知给app让其任务下载继续，当这个app正在下载的时候突然退出，这个任务将要继续下载。

当任务完成的时候，这个守护进程将要重新加载在后台模式中，这个重新加载app将要重新连接这同样的session.收到相关的完成的代理消息并且执行一些要求的动作，比如持久化下载的文件到硬盘上等。

>*注意*
>如果你强制退出app通过app switche，这个系统将要取消所有后台下载的任务，并且不会再视图重启这个app
>

仍然在SearchViewController.swift这个文件中，在初始化 `downloadsSession `的地方，找到下面这行代码:

```
let configuration = NSURLSessionConfiguration.defaultSessionConfiguration()

```
替换成下面的代码:

```
let configuration = NSURLSessionConfiguration.backgroundSessionConfigurationWithIdentifier("bgSessionConfiguration")
```
替换默认的session配置文件，指定一个特殊的后台session配置文件，注意你设置了唯一的ID为这个session,这会允许你引用并且重新连接同样的后台session.

下一步，在 viewDidLoad()中，增加如下代码:

```
_ = self.downloadsSession
```

调用懒加载属性`downloadsSession `,确保应用程序确实创建了一个后台session 的SearchViewController的实例。




当一个后台任务完成的时候，这个App不再运行，这个app将会重新运行到后台进程中，你需要去处理你的app的一些代理方法.

切换到 AppDelegate.swift,添加下面的代码在类的顶部:

```
var backgroundSessionCompletionHandler: (() -> Void)?
```

下一步，添加如下代码在AppDelegate.swift:

```
func application(application: UIApplication, handleEventsForBackgroundURLSession identifier: String, completionHandler: () -> Void) {
  backgroundSessionCompletionHandler = completionHandler
}
```

提供一个completionHandler作为一个变量在你的App代理方法中，等会会用到。


application(_:handleEventsForBackgroundURLSession:)会唤醒这个App处理完成这个后台任务，你需要去处理两个事情：

* 首先，通过代理方法用这个App去重新连接这个后台session,一旦你创建并且每次使用后台session时，SearchViewController就会被实例化，你已经重新连接了.
* 第二，你需要去捕获完成的回调方法，在完成的回调函数中，更新你的UI,然后告诉系统你的App已经工作完毕使用后台任务的session.

但是什么时候你将会调用完成的回调函数呢？
`URLSessionDidFinishEventsForBackgroundURLSession(_:) `将会是一个好的选择，它是NSURLSessionDelegate的一个代理方法，当所有的任务在后台session中完成的时候。


实现下面的扩展在`SearchViewController.swift `中

```
extension SearchViewController: NSURLSessionDelegate {
 
  func URLSessionDidFinishEventsForBackgroundURLSession(session: NSURLSession) {
    if let appDelegate = UIApplication.sharedApplication().delegate as? AppDelegate {
      if let completionHandler = appDelegate.backgroundSessionCompletionHandler {
        appDelegate.backgroundSessionCompletionHandler = nil
        dispatch_async(dispatch_get_main_queue(), {
          completionHandler()
        })
      }
    }
  }
}
```
上面的代码仅仅抓取了在APPDelegate中存储的回调函数，并且在主线中调用它.

编译运行你的工程，开始几个下载之后迅速按在home键，使得下载任务进入后台，等几十秒你的下载任务将会完成，然后双击home,关闭当前程序。

下载任务将会完成并且他们将会更新显示状态。

![logo](http://7xkxhx.com1.z0.glb.clouddn.com/Simulator-Screen-Shot-19-Aug-2015-1.06.24-am-281x500.png)

OK，这个demo已经完备了。

##完整工程下载
你可以下载完整的工程从 [这里](http://www.raywenderlich.com/wp-content/uploads/2016/01/HalfTunes-Final.zip)



更多资源

* [苹果的官方文档](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSession_class/)
* [AlamoFire](https://github.com/Alamofire/Alamofire)是Swift中非常流行的第三方框架，可以学习一下.






