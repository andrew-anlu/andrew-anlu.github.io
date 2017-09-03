---
layout: post
title: "ios10 UserNotifications"
date: 2016-10-17 09:53:08 +0800
comments: true
categories: iOS
---

ios10以前杂乱的和通知相关的API都被统一了，现在开发者可以使用独立的UserNotificaitons.framework来集中管理和使用iOS系统中通知的功能。在此基础上，Apple还增加了撤回单条通知，更新已展示通知，中途修改通知内容，在通知中展示图片视频，自定义通知UI等一系列功能。

<!--more-->

对于开发者来说，想比较于之前版本，iOS10提供了一套非常易用通知处理接口，是SDK的一次重大重构，而之前的绝大部分通知相关API都已经被标为弃用(deprecated)

您可以在WWDC16的 [introducaion to Notifications](https://developer.apple.com/videos/play/wwdc2016/707/)和[Advanced Notifications](https://developer.apple.com/videos/play/wwdc2016/708/)这两个Session中找到详细信息；另外也不要忘了参考[UserNotifications官方文档](https://developer.apple.com/reference/usernotifications)



## UserNOtifications框架解析

### 基本流程
iOS10中通知相关的操作遵循下面的流程:

`审核和注册`->`创建和发起`->`展示和处理`


首先你需要向用户请求推送权限，然后发送通知。对于发送出的通知，如果你的应用位于后台或者没有运行的话，系统将通过用户允许的方法（弹窗，横幅，或者是在通知中心）进行展示。如果你的应用已经位于前台正在运行，你可以自行决定要不要显示这个通知。最后，如果你希望用户点击通知能有打开应用以外的额外功能的话，你也需要进行处理。

##权限申请
iOS8之前，本地推送和远程推送（Remote Notificaiton）是区分对待的，应用只需要在进行远程推送是获取用户同意。iOS8对这一行为进行了规范，因为无论是本地推送还是远程推送，其实在用户看来表现是一致的，都是打断用户的行为。因此从iOS8开始，这两种通知都需要申请权限。ios10里进一步消除了本地通知和推送通知的区别。向用户申请通知权限非常简单：

```
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) {
    granted, error in
    if granted {
        // 用户允许进行通知
    }
}
```

当然，在使用UN开头的API的时候，不要忘记导入`UserNotifications `框架：

`import UserNotifications`

第一次调用这个方法时，会弹出一个系统弹窗.

![1](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20161017-0.png)

要注意的是，一旦用户拒绝了这个请求，再次调用该方法时也不会进行弹窗，想要应用有机会接收到通知的话，用户必须自行前往系统的设置中为你的应用打开通知，而这往往是不可能的。因此，在合适的是偶弹出请求窗，在请求权限前预先进行说明，而不是直接粗暴地在启动的时候就进行弹窗，会是更明智的选择。

## 远程推送
一旦用户同意后，你就可以再应用中发送本地通知了。不过如果你通过服务器发送远程通知的话，还需要多一个获取用户token的操作，你的服务器可以使用这个token将用向Apple push Notification的服务器提交请求，然后APNS通过token识别设备和应用，将通知推给用户。

提交token请求和获得token的回调是现在"唯一"不在新框架中的API,我们使用`UIApplication`的`registerForRemoteNotifications `来注册远程通知，在`AppDelegate`的`application(_:didRegisterForRemoteNotificationsWithDeviceToken)`中获取用户token:

```
// 向 APNs 请求 token：
UIApplication.shared.registerForRemoteNotifications()

// AppDelegate.swift
 func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    let tokenString = deviceToken.hexString
    print("Get Push token: \(tokenString)")
}
```

获取得到的`deviceToken `是一个`Data`类型，为了方便使用和传递，我们一般会选择将它转换为一个字符串。swift3中可以使用下面的`data`扩展来构造适合传递给Apple的字符串：

```
extension Data {
    var hexString: String {
        return withUnsafeBytes {(bytes: UnsafePointer<UInt8>) -> String in
            let buffer = UnsafeBufferPointer(start: bytes, count: count)
            return buffer.map {String(format: "%02hhx", $0)}.reduce("", { $0 + $1 })
        }
    }
}
```

### 权限设置
用户可以再系统设置中修改你的应用的通知权限，除了打开和关闭全部通知权限外，用户也可以限制你的应用智能进行某种形式的通知显示，比如值允许横幅而不允许弹窗及通知中心显示灯。一般来说你不应该对用户的选择进行干涉，但是如果你的应用确实需要某种特定场景的推送的话，你可以对当前用户进行的设置进行检查:

```
UNUserNotificationCenter.current().getNotificationSettings {
    settings in 
    print(settings.authorizationStatus) // .authorized | .denied | .notDetermined
    print(settings.badgeSetting) // .enabled | .disabled | .notSupported
    // etc...
}
```


## 发送通知

UserNOtifications中对通知进行了统一。我们通过通知的内容（`UNNotificaitonsContent`）,发送的时机`UNNotifiationTrigger`以及一个发送通知的`String`类型的标识符，来生成一个`UNNotificationRequest `类型的发送请求。最后，我们将这个请求添加到`UNUserNotificationCenter.current()`中，就可以等待通知到达了：

```
// 1. 创建通知内容
let content = UNMutableNotificationContent()
content.title = "Time Interval Notification"
content.body = "My first notification"

// 2. 创建发送触发
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)

// 3. 发送请求标识符
let requestIdentifier = "com.onevcat.usernotification.myFirstNotification"

// 4. 创建一个发送请求
let request = UNNotificationRequest(identifier: requestIdentifier, content: content, trigger: trigger)

// 将请求添加到发送中心
UNUserNotificationCenter.current().add(request) { error in
    if error == nil {
        print("Time Interval Notification scheduled: \(requestIdentifier)")
    }
}
```

1. iOS10中通知不仅支持简单的一行文字，你还可以添加`title`和 `subtitle`,来用粗体字的形式强调通知的目的。对于远程推送，iOS10之前一般只含有消息的推送；payload是这样的：

```
{
  "aps":{
    "alert":"Test",
    "sound":"default",
    "badge":1
  }
}
```
如果我们想要加入`title`和`subtitle`的话，则需要将`alert`从字符串换为字典，新的payload是:

```

  "aps":{
    "alert":{
      "title":"I am title",
      "subtitle":"I am subtitle",
      "body":"I am body"
    },
    "sound":"default",
    "badge":1
  }
}
```

好消息是，后一种字典的方法其实在iOS8.2的时候就已经存在了，虽然当时`title`只是用在Apple Watch上的，但是设置好`body`的话在iOS上还是可以显示的，所以针对iOS10添加标题时是可以保证向前兼容的。

另外，如果要进行本地化对应，在设置这些内容文本时，本地可以使用`String.localizedUserNotificationString(forKey: "your_key", arguments: []) `的方式来从`Localizable.strings`文件中取出本地化字符串，而远程推送的话，也可以再payload的alert中使用`loc-key`或者`title-loc-key`来进行指定

2. 触发器是只对本地通知而言的，远程推送的通知的话默认会在收到后立即显示。现在`UserNotifications`框架中提供了三种触发器，分别是：在一定时间后触发`UNTimeIntervalNotificationTrigger `,在某月某日某时触发`UNCalendarNotificationTrigger `,以及在用户进入或者离开某个区域时触发`UNLocationNotificationTrigger `

3. 请求标识符可以用来区分不同的通知请求，在将一个通知请求提交后，通过特定的API我们能够使用这个标识符来取消或者更新这个通知。我们将在稍后提到具体方法

4. 在新版本的通知框架中，Apple借用了一部分网络请求的概念，我们组织并发送一个通知请求，然后将这个请求提交给`UNUserNotificationCenter `进行处理。我们会在delegate中接收到这个通知请求对应的responst,另外我们也有机会再应用的extension中对request进行处理

在提交请求后，我们锁屏或者将应用切到后台，并等待设定的时间后，就能看到我们的通知出现在通知中心或者屏幕横幅了:
![1](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20161017-1.png)


## 取消和更新
在创建通知请求时，我们已经制定了标识符。这个标识符可以用来管理通知，在iOS10之前，我们很难取消掉某一个特定的通知，也不能主动移除或者更新已经展示的通知。

iOS10中，UserNotifications框架提供了一系列管理通知的API,你可以做到:

* 取消还未展示的通知
* 更新还未展示的通知
* 移除已经展示过的通知
* 更新已经展示过的通知

其中关键就是创建请求时使用同样的标识符：

比如，从通知中心移除一个展示过得通知：

```
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 3, repeats: false)
let identifier = "com.onevcat.usernotification.notificationWillBeRemovedAfterDisplayed"
let request = UNNotificationRequest(identifier: identifier, content: content, trigger: trigger)

UNUserNotificationCenter.current().add(request) { error in
    if error != nil {
        print("Notification request added: \(identifier)")
    }
}

delay(4) {
    print("Notification request removed: \(identifier)")
    UNUserNotificationCenter.current().removeDeliveredNotifications(withIdentifiers: [identifier])
}
```
类似的，我们可以使用`removePendingNotificationRequests `，来取消还未展示的通知请求。对于更新通知，不论是否已经展示，都和一开始添加请求时一样，再次将请求提交给`UNUserNotificationCenter `即可：

```
// let request: UNNotificationRequest = ...
UNUserNotificationCenter.current().add(request) { error in
    if error != nil {
        print("Notification request added: \(identifier)")
    }
}

delay(2) {
    let newTrigger = UNTimeIntervalNotificationTrigger(timeInterval: 1, repeats: false)

    // Add new request with the same identifier to update a notification.
    let newRequest = UNNotificationRequest(identifier: identifier, content:newContent, trigger: newTrigger)
    UNUserNotificationCenter.current().add(newRequest) { error in
        if error != nil {
            print("Notification request updated: \(identifier)")
        }
    }
}
```

远程推送可以进行通知的更新，在使用Provider API向APNS提交请求时，在HTTP2的header中`apns-collapse-id`key的内容将被作为该推送的标识符进行使用。多次推送同一标识符的通知即可进行更新

##处理通知
### 应用内展示通知
现在系统可以在应用处于后台或者退出的时候向用户展示通知了。不过，当应用处于前台时，收到的通知是无法进行展示的。如果我们希望在应用内也能显示通知的话，需要额外的工作。

`UNUserNotificationCenterDelegate `提供了两个方法，分别对应如何在应用内展示通知，和收到通知响应时要如何处理的工作。我们可以实现这个借口中的对应方法来在应用内展示通知：

```
class NotificationHandler: NSObject, UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, 
                       willPresent notification: UNNotification, 
                       withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) 
    {
        completionHandler([.alert, .sound])

        // 如果不想显示某个通知，可以直接用空 options 调用 completionHandler:
        // completionHandler([])
    }
}
```

实现后，将`NotificationHandler `的实例赋值给`UNUserNotificationCenter `的`delegate`属性就可以了。没有特殊理由的话，AppDelegate的`application(_:didFinishLaunchingWithOptions:)`就是一个不错的选择：

```
class AppDelegate: UIResponder, UIApplicationDelegate {
    let notificationHandler = NotificationHandler()
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        UNUserNotificationCenter.current().delegate = notificationHandler
        return true
    }
}
```

## 对通知进行响应
`UNUserNotificationCenterDelegate `中还有一个方法,`userNotificationCenter(_:didReceive:withCompletionHandler:)`这个代理方法会在用户与你推送的通知进行交互时被调用，包括用户通过通知打开了你的应用，或者点击或者触发了某个Action。因为涉及到打开应用的行为，所以事先了这个方法的delegate必须在`applicationDidFinishLaunching:`返回前就完成设置，这也是我们之前推荐将`NotificationHandler `今早进行赋值的理由。

一个最简单的事先自然什么也不错，直接告诉系统你已经完成了所有工作。

```
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    completionHandler()
}
```

在该方法里，我们将获取到这个推送请求对应的response,`UNNotificationResponse `是一个几乎包括了通知的所有信息的对象，从中我们可以再次获取到`userInfo `中的信息:

```
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    if let name = response.notification.request.content.userInfo["name"] as? String {
        print("I know it's you! \(name)")
    }
    completionHandler()
}
```

更好的消息是，远程推送的payload内的内容也会出现在这个`userInfo`中，这样一来，不论是本地推送还是远程推送，处理的路径得到了统一。通过`userInfo`的内容来决定页面跳转或者是进行其他操作，都会有很大空间。

## Actionable通知发送和处理
### 注册Category
iOS8和9中Apple引入了可以交互的通知，这是通过将一簇action放到了一个category中，将这个category进行注册，最后在发送通知时将通知的category设置为要使用的category来实现的。

![1](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20161017-2.png)

注册一个category非常容易：

```
private func registerNotificationCategory() {
    let saySomethingCategory: UNNotificationCategory = {
        // 1
        let inputAction = UNTextInputNotificationAction(
            identifier: "action.input",
            title: "Input",
            options: [.foreground],
            textInputButtonTitle: "Send",
            textInputPlaceholder: "What do you want to say...")

        // 2
        let goodbyeAction = UNNotificationAction(
            identifier: "action.goodbye",
            title: "Goodbye",
            options: [.foreground])

        let cancelAction = UNNotificationAction(
            identifier: "action.cancel",
            title: "Cancel",
            options: [.destructive])

        // 3
        return UNNotificationCategory(identifier:"saySomethingCategory", actions: [inputAction, goodbyeAction, cancelAction], intentIdentifiers: [], options: [.customDismissAction])
    }()

    UNUserNotificationCenter.current().setNotificationCategories([saySomethingCategory])
}
```

1. `UNTextInputNotificationAction `代表一个输入文本的action,你可以自定义框的按钮title和placeholder,你稍后会使用`identifier `来对action进行区分。
2. 普通的`UNNotificationAction `对应标准的按钮
3. 为category指定一个`identifier`,我们将在实际发送通知的时候用这个标识符进行设置，这样系统就知道对应哪个category了。

当然，不要忘了在程序启动时调用这个方法进行注册

```
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    registerNotificationCategory()
    UNUserNotificationCenter.current().delegate = notificationHandler
    return true
}
```


### 发送一个带有action的通知
在完成category注册后，发送一个actionable通知就非常简单了，只需要在创建`UNNotificationContent `时把`categoryIdentifier `设置为需要的categoryId即可：

```
content.categoryIdentifier = "saySomethingCategory"
```

尝试展示这个通知，在下拉或者使用3D touch展开通知后，就可以看到对应的action了：

![1](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20161017-3.png)

远程推送也可以使用category,只需要在payload中添加`category`字段，并指定预先定义的category id 就可以了:

```
{
  "aps":{
    "alert":"Please say something",
    "category":"saySomething"
  }
}
```

### 处理actionable通知
和普通的通知并无二致力，actionable通知也会走到`didReceive`的delegate方法，我们通过request中包含的`categoryIdentifier `和response里的`actionIdentifier `就可以轻易判定是那个通知的那个操作被执行了。对于`UNTextInputNotificationAction `触发的response,直接将它转换为一个`UNTextInputNotificationResponse `，就可以拿到其中的用户输入了：

```
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {

    if let category = UserNotificationCategoryType(rawValue: response.notification.request.content.categoryIdentifier) {
        switch category {
        case .saySomething:
            handleSaySomthing(response: response)
        }
    }
    completionHandler()
}

private func handleSaySomthing(response: UNNotificationResponse) {
    let text: String

    if let actionType = SaySomethingCategoryAction(rawValue: response.actionIdentifier) {
        switch actionType {
        case .input: text = (response as! UNTextInputNotificationResponse).userText
        case .goodbye: text = "Goodbye"
        case .none: text = ""
        }
    } else {
        // Only tap or clear. (You will not receive this callback when user clear your notification unless you set .customDismissAction as the option of category)
        text = ""
    }

    if !text.isEmpty {
        UIAlertController.showConfirmAlertFromTopViewController(message: "You just said \(text)")
    }
}
```

上面的代码先判断通知响应是否属于`saySomething`，然后从用户输入或者是选择中提取字符串，并且弹出一个alert作为响应结果。当然，更多请苦情下我们会发送一个网络请求，或者是根据用户操作更新一些UI等。

## Notificiaton Extension
iOS10中添加了很多extention,作为应用与系统整合的入口。与通知相关的extension有两个：Service Extension和Content Extension.前者可以让我们有机会在收到远程推送的通知后，展示之前对通知内容进行修改：后者可以用来自定义通知视图的样式。

![1](http://7xkxhx.com1.z0.glb.clouddn.com/notification-extensions.png)

### 截取并修改通知内容
`NotificationService `的模板已经为我们进行了基本的实现:

```
class NotificationService: UNNotificationServiceExtension {

    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?

    // 1
    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)

        if let bestAttemptContent = bestAttemptContent {
            if request.identifier == "mutableContent" {
                bestAttemptContent.body = "\(bestAttemptContent.body), Andrew"
            }
            contentHandler(bestAttemptContent)
        }
    }

    // 2
    override func serviceExtensionTimeWillExpire() {
        // Called just before the extension will be terminated by the system.
        // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
        if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }
}
```

1. `didReceive:`方法中有一个等待发送的通知请求，我们通过修改这个请求中的content内容，然后在限制的时间内将修改后的内容调用通过`contentHandler `返还给系统，就可以显示这个修改过得通知了
2. 在一定时间内没有调用`contentHandler `的话，系统会调用这个方法，来告诉你大限已到。你可以选择什么都不做，这样的话系统将当做什么都没发生，简单地显示原来的通知，可能你其实已经设置好了绝大部分内容，只是有很少一部分没有完成，这时你也可以像例子中这样调用`contentHandler `来显示一个变更"中途"的通知

Service Extentsion现在只对远程推送的通知起效，你可以在推送payload中增加一个`mutable-content`的值为1的项来启用内容修改：

```
{
  "aps":{
    "alert":{
      "title":"Greetings",
      "body":"Long time no see"
    },
    "mutable-content":1
  }
}
```

这个payload的推送得到的结果就是推送的内容+“Andrew”

使用在本机截取推送并替换内容的方式，可以完成端到端(end-to-end)的推送加密。你在服务器推送payload中加入加密过得文本，在客户端接到通知后使用预先定义或者获取过得秘钥进行解密，然后立即显示。这样一来，即使推送信道被第三方截取，其中所传递的内容也还是安全的。使用这种方式来发送密码或者敏感信息，对于一些金融业务应用和聊天应用来说，应该是必备的特性。


## 在通知中展示图片/视频
相比于旧版本的通知，iOS10中另一个亮眼功能室多媒体的推送，开发者现在可以在通知中嵌入图片或者视频，这极大丰富了推送内容的可读性和趣味性。

为本地通知添加多媒体内容十分简单，只需要通过本地磁盘上的文件URL创建一个`UNNotificationAttachment `对象，然后将这个对象放到数组中赋值给`content`的`attachments`属性就行了：

```
let content = UNMutableNotificationContent()
content.title = "Image Notification"
content.body = "Show me an image!"

if let imageURL = Bundle.main.url(forResource: "image", withExtension: "jpg"),
   let attachment = try? UNNotificationAttachment(identifier: "imageAttachment", url: imageURL, options: nil)
{
    content.attachments = [attachment]
}
```

在显示时，横幅或者弹窗将附带设置的图片，使用3D Touch pop通知或者下拉通知显示详细内容时，图片也会被放大显示：

![2](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20161017-4.png)

除了图片之外，通知还支持音频以及视频。你可以将MP3或者MP4这样的文件提供给系统来在通知中进行展示和播放，不过，这些文件都有尺寸的限制，比如图片不能超过5MB,视频不能超过50MB,不过对于一般的能在通知中展示的内容来说，这个尺寸应该是绰绰有余了。关于支持的文件格式和尺寸，可以在文档中进行确认。在创建`UNNotificationAttachment `时，如果遇到了不支持的格式，SDK也会抛出错误。

通过远程推送的方式，你也可以显示图片等多媒体内容，这要借助于上一节所提到的通过`Notification Service Extension `来修改涂松通知内容的技术。一般做法是，我们在推送payload中指定需要加载的图片资源地址，这个地址可以是应用bundle内已经存在的资源，也可以是网络的资源。不过因为在创建`UNNotificationAttachment `时偶们只能使用本地资源，所以如果多媒体还不在本地的话，我们需要先将其下载到本地，在完成`UNNotificationAttachment `创建后，我们就可以和本地通知一样，将它设置给`attachments `属性，然后调用`contentHandler `了。

简单的实例 payload如下：

```
{
  "aps":{
    "alert":{
      "title":"Image Notification",
      "body":"Show me an image from web!"
    },
    "mutable-content":1
  },
  "image": "https://onevcat.com/assets/images/background-cover.jpg"
}
```

`mutable-content`表示偶们会在接收到通知时对内容进行更改，`image`指明了目标图片的地址。

在`NotificationService `里，加入如下代码来下载图片，并将其保存到磁盘缓存中：

```
private func downloadAndSave(url: URL, handler: @escaping (_ localURL: URL?) -> Void) {
    let task = URLSession.shared.dataTask(with: url, completionHandler: {
        data, res, error in

        var localURL: URL? = nil

        if let data = data {
            let ext = (url.absoluteString as NSString).pathExtension
            let cacheURL = URL(fileURLWithPath: FileManager.default.cachesDirectory)
            let url = cacheURL.appendingPathComponent(url.absoluteString.md5).appendingPathExtension(ext)

            if let _ = try? data.write(to: url) {
                localURL = url
            }
        }

        handler(localURL)
    })

    task.resume()
}
```
然后再`didReceive: `中，接收到这类通知时提取图片地址，下载，并生成attachment,进行通知展示：

```
if let imageURLString = bestAttemptContent.userInfo["image"] as? String,
   let URL = URL(string: imageURLString)
{
    downloadAndSave(url: URL) { localURL in
        if let localURL = localURL {
            do {
                let attachment = try UNNotificationAttachment(identifier: "image_downloaded", url: localURL, options: nil)
                bestAttemptContent.attachments = [attachment]
            } catch {
                print(error)
            }
        }
        contentHandler(bestAttemptContent)
    }
}
```
关于在通知中展示图案品或者视频，有几点想补充说明：

1. `UNNotificationContent `的`attachments `虽然是一个数组，但是系统只会展示第一个attachmen对象的内容。不过你依然可以发送多个`attachments `,然后再要展示的时候再重新安排他们的顺序，以显示最符合情景的图片或者视频。另外，你也可能会在自定义通知展示UI时用到多个`attachment `,我们接下来一节中会看到一个相关的例子。
2. 在当前iOS10中，`serviceExtensionTimeWillExpire `被条用之前，你有30秒时间来处理和更改通知内容，对于一般的图片来说，这个时间是足够的，但是如果你推送的体积较大的视频内容，用户又恰巧在糟糕的网络环境的话，很有可能无法及时下载完成。
3. 如果你想在远程推送来的通知中显示应用bundle内的资源的话，要注意extension的bundle和app main bundle并不是一回事，你可以选择将图片资源放到extension bundle中，也可以选择放在main bundle里，总之，你需要保证能够获取到正确的，并且你具有读取权限的url
4. 系统在创建`attachement `时会根据提供的url后缀确定文件类型，如果没有后缀，或者后缀无法不正确的话，你可以在创建时通过`UNNotificationAttachmentOptionsTypeHintKey `来[指定资源类型](https://developer.apple.com/reference/usernotifications/unnotificationattachmentoptionstypehintkey)
5. 如果使用的图片和视频文件不在你的bundle内部，它们将被移动到系统的负责通知的文件夹下，然后当同志被移除后删除。如果媒体文件在bundle内部，它们将被负责到通知文件夹下。每个应用能使用的媒体文件大小总和是有限制，超过限制后创建`attachment `时将抛出异常。可能的所有错误可以再`UNError`中找到
6. 你可以访问一个已经创建的`attachment `的内容，但是要注意权限问题，可以使用`startAccessingSecurityScopedResource `来暂时获取以创建的`attachment `的访问权限。比如：

```
let content = notification.request.content
if let attachment = content.attachments.first {  
    if attachment.url.startAccessingSecurityScopedResource() {  
        eventImage.image = UIImage(contentsOfFile: attachment.url.path!)  
        attachment.url.stopAccessingSecurityScopedResource()  
    }  
}  
```

## 自定义通知视图样式
ioS10 SDK 新加的另一个`Content Extension `可以用来自定义通知的详细页面的视图，新建一个`Notification Content Extension`,Xcode为我们准备的模板中包含了一个实现了`UNNotificationContentExtension `的`UIViewController `子类。这个extension中有个一必须实现的方法`didReceive(_:)`,在系统需要显示自定义样式的通知详情视图时，这个方法将被调用，你需要在其中配置你的UI.而UI本身可以通过这个extension中的`MainInterface.storyboard`来进行定义。自定义UI的通知是和通知category绑定的，我们需要在`extension `的info.plist里指定这个通知样式所对应的category标识符：

![1](http://7xkxhx.com1.z0.glb.clouddn.com/notification-content-info.png)

系统在接收到通知后会先查找有没有能够处理这类通知的content extension,如果存在，那么就交给extensionl来进行处理，另外，在构建UI时，我们可以通过Info.plist控制通知详细视图的尺寸，以及是否显示原始的通知。关于Content Extension中的info.plist的key,可以在[这个文档](https://developer.apple.com/reference/usernotificationsui/unnotificationcontentextension)中找到详细信息。

虽然我们可以使用包括按钮在内的各种UI，但是系统不允许我们队这些UI进行交互，点击通知视图UI本身会将我们导航到应用中，不过我们可以通过action的方式来对自定义UI进行更新。`UNNotificationContentExtension `为我们提供了一个可选方法`didReceive(_:completionHandler:)`，它会在用户选择了某个action时被调用，你有机会在这里更新通知的UI，如果有UI更新，那么在方法的`completionHandler `中，开发者可以选择传递`. doNotDismiss `来保持通知继续呗显示。如果没有继续显示的必要，可以选择`. dismissAndForwardAction `或者`. dismiss `，前者将把通知的action继续传递给应用的`UNUserNotificationCenterDelegate `中的`userNotificationCenter(:didReceive:withCompletionHandler)`，而后者将直接解散这个通知


如果你的自定义UI包含视频等，你还可以实现`UNNotificationContentExtension `里的`media `开头的一系列属性，它将为你提供一些视频播放的控件和相关方法。


## 总结
iOS10 SDk中对通知这块进行了IOS系统发布以来最大的一次重构，很多"老朋友"都被标记为了 deprecated:

## iOS10中被标记弃用的API

* UILocalNotification
* UIMutableUserNotificationAction
* UIMutableUserNotificationCategory
* UIUserNotificationAction
* UIUserNotificationCategory
* UIUserNotificationSettings
* handleActionWithIdentifier:forLocalNotification:
* handleActionWithIdentifier:forRemoteNotification:
* didReceiveLocalNotification:withCompletion:
* didReceiveRemoteNotification:withCompletion:

等一系列在`UIKIT`中的发送和处理通知的类型及方法


相比较于iOS早期时代的API,新的APi展现了高度的模块化和统一特性，易用性也非常好，是一套更加先进的API,如果有可能，特别是如果你的应用是重度依赖通知特性的话，直接从iOS10开始可以让你充分使用在新同志体系的各种特性。

虽然原来的API都被标为弃用了，但是如果需要支持iOS10之前系统的话，你还是需要使用原来的API,我们可以使用

```
if #available(iOS 10.0, *) {
    // Use UserNotification
}
```
的方式来对iOS10进行新通知的适配，并让iOS10的用户享受到新通知带来的便利特性，然后在将来版本升级到只支持iOS10以上时再移除掉所有被启用的代码。对于优化和梳理通知相关代码来说，新API对代码设计和祖上上带来的好处足以弥补适配上的麻烦。

