---
layout: post
title: "用CoreLocation实现-地理围栏"
date: 2016-12-19 14:58:38 +0800
comments: true
categories: swift
---

地理围栏就是当你的手机设备进入或者离开某个区域的时候进行消息提醒,它会让你的程序变得更cool,设想一下，当你离开家，或者靠近一家你喜欢的商场附近时，能够及时给你发送最新的或者最优惠的信息。


<!--more-->

在这个教程中，你将会学会如何用region监听区域，这个工程是采用swift3语言实现的。

为了练习，你将要创建一个位置点去提醒app调用地理围栏进行提醒，假想它们就是现实世界中的位置坐标。好了，让我们开始吧。


## 开始

下载这个[开始工程](https://koenig-media.raywenderlich.com/uploads/2016/09/Geotify-Starter-1.zip),这个工程提供了简单的在地图上添加/删除 地图注解(Annotation)的方法

编译运行程序，你将会看到一个空白的地图:

![1](https://koenig-media.raywenderlich.com/uploads/2016/06/GeoInitial-281x500.png)

点击`+`按钮，去添加一个新的地理围栏，首先会打开一个新的页面，你可以设置你想要的位置坐标，点击`Add`就添加到了地图上，并且大头针显示

![2](https://koenig-media.raywenderlich.com/uploads/2016/06/GeoLooking-Around-281x500.png)

这个`radius`代表指定地点的半径范围，单位是米
`note`在导航过程中，能够展示你想要展示的任何内容。

这个App也能够设置触发提醒方式，当用户进入或者离开某个区域的时候，进行提醒。

对radius设置1000，并且设置note为`Say Hi to Tim!`,选择触发类型为`Upon Entry`,然后点击`Add`

然后你将要看到的地理围栏已经出来了一个新的大头针在地图上，下面还有一个圆形的渲染图层

![2](https://koenig-media.raywenderlich.com/uploads/2016/06/Geo-Say-Hi-281x500.png)

点击这个大头针，将会显示详细信息，比如提示信息和半径大小，不要点击左边的删除按钮，因为点击之后这个地理围栏信息将会被删除掉从地图上和内存中

>*注意:*
>所有的地理围栏的信息都存储到了NSUserDefaults中
>
>
>


## 设置 Location Manager和权限

你添加到地图上的地理围栏仅仅是能看的，但是不能进行消息提醒。

你将要修复这个通过core location来进行监听

打开`GeotificationsViewController.swift `，然后在类的顶部定义一个常量实例`CLLocationManager `:

```
class GeotificationsViewController: UIViewController {
 
  @IBOutlet weak var mapView: MKMapView!
 
  var geotifications = [Geotification]()
  let locationManager = CLLocationManager() // Add this statement
 
  ...
}
```

下一步，替换`viewDidLoad()`中为下面的代码:

```
override func viewDidLoad() {
  super.viewDidLoad()
  // 1
  locationManager.delegate = self
  // 2
  locationManager.requestAlwaysAuthorization()
  // 3
  loadAllGeotifications()
}
```

我们解析下代码:

1. 设置locationManager的代理，只有设置了代理，相关的代理方法才会被调用
2. 你调用了`requestAlwaysAuthorization()`方法，它会调用一个提示，"Always"总是允许使用定位服务，因为App一直需要拥有`Always`权限去进行地理围栏的监听，直到这个app不在运行的时候，就不再需要这个权限了。在info.plist中已经设置了消息去告诉用户当请求定位信息时必须设置key:`NSLocationAlwaysUsageDescription `
3. 调用` loadAllGeotifications()`，转换存储在`NSUserDefaults `中的地理围栏的信息，然后加载它们，这个方法加载地图上所有的地理围栏和大头针的数据


当app设置了用户权限之后，界面上将会显示`NSLocationAlwaysUsageDescription `提示，一个友好的提示为什么app需要请求用户的地理坐标。这个key是强制性假如你使用定位服务的话，如果这个key没有的话，系统将会忽略程序请求并且终止定位服务

![2](https://koenig-media.raywenderlich.com/uploads/2016/06/GeoLocationWhenNotUsing-281x500.png)


OK，你已经设置好了app的请求权限，点击`Allow`去允许` location manager`在合适的时机调用代理方法

在你实现地理围栏的提醒之前，这里有个小的问题你必须解决：用户的当前坐标没有展示在地图上，这个特性是不能实现的，你必须手动的点击左上角的那个定位按钮，才能定位到当前用户的地理坐标。

幸运的是，修复这个是很容易的-在你允许了app获取权限之后，仅仅需要点下那个定位按钮

在`GeotificationsViewController.swift `，添加`CLLocationManagerDelegate `的扩展如下:

```
extension GeotificationsViewController: CLLocationManagerDelegate {
  func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
    mapView.showsUserLocation = (status == .authorizedAlways)
  }
}
```


不管什么时候用户的权限状态改变的时候，location manager都会调用`locationManager(_:didChangeAuthorizationStatus:)`代理方法。假如用户授权app去使用当前定位服务，在你初始化好后locationManager,并且设置了代理之后，这个方法将会被调用


这个方法是个理想的地方去检查这个app是否被授权，如果是，你将能确保mapView展示当前用户的坐标

编译运行这个app,如果你用真机测试，你将会在mapView中看到当前的坐标；如果你运行在模拟器上，点击 `Debug\Location\Apple` 来查看地图上的标记

![1](https://koenig-media.raywenderlich.com/uploads/2016/06/GeoLocationFar-281x500.png)

放大地方，你看到的将会是这样:

![1](https://koenig-media.raywenderlich.com/uploads/2016/06/GeoLocationZoomed-281x500.png)


## 注册你自己的地理围栏

当你的location manager配置好了之后，下一步就是允许你的app去注册一个地理围栏用来被监听


在你的app中，你的地理围栏的信息是被存储为`Geotification `模型，在你注册为被监听之前，core Location 请求每一个地理围栏去返回一个`CLCircularRegion `实例。为了去处理这个请求，你将要创建一个helper方法，然后从指定的`Geotification `对象中返回一个  `CLCircularRegion`

打开`GeotificationsViewController.swift`，然后在主体代码中添加如下方法:

```
func region(withGeotification geotification: Geotification) -> CLCircularRegion {
  // 1
  let region = CLCircularRegion(center: geotification.coordinate, radius: geotification.radius, identifier: geotification.identifier)
  // 2
  region.notifyOnEntry = (geotification.eventType == .onEntry)
  region.notifyOnExit = !region.notifyOnEntry
  return region
}
```

上面的方法做的工作如下：

1. 你用地理围栏的坐标进行`CLCircularRegion `的初始化，这个地理围栏的半径和identifier允许ios去判断注册的地理围栏的距离，这个初始化是很简单的，`Geotification `模型已经包含了要请求的属性。
2. `CLCircularRegion `实例也需要设置两个BOOL值得属性，`notifyOnEntry `和`notifyOnExit `,这是两个标识，当设备进入或者离开指定的地理围栏的时候，定义的地理围栏的回调事件将会被触发，你也可以为你的每个地理围栏设计去响应一个消息通知，你可以设置一个是true,另外一个是false,前提是你需要使用`Geotification `实体的枚举值


下一步，当用户添加坐标的时候，你需要一个方法去开始监听这个地理围栏

添加下面的方法在`GeotificationsViewController `：

```
func startMonitoring(geotification: Geotification) {
  // 1
  if !CLLocationManager.isMonitoringAvailable(for: CLCircularRegion.self) {
    showAlert(withTitle:"Error", message: "Geofencing is not supported on this device!")
    return
  }
  // 2
  if CLLocationManager.authorizationStatus() != .authorizedAlways {
    showAlert(withTitle:"Warning", message: "Your geotification is saved but will only be activated once you grant Geotify permission to access the device location.")
  }
  // 3
  let region = self.region(withGeotification: geotification)
  // 4
  locationManager.startMonitoring(for: region)
}
```

让我们一步一步讲解上面的方法:

1. `isMonitoringAvailableForClass(_:)`方法决定我们的设备是否支持监听地理围栏，假如不能够监听，程序将会返回并且弹出一个提示框进行提醒。
2. 下一步，你检查当前权限的状态，确保这个app已经被授权去请求用户的定位服务，加入这个app没有被授权，这个设备将不会接受到地理围栏的任何提示信息。然而，在这个案例中，你将要始终允许用户去保存地理围栏信息，因为当你的app没有权限的时候，core loaction会让你注册地理围栏。当用户权限给这个app的时候，监听这些地理围栏将会自动开启
3. 在之前定义的方法中，你创建一个`CLCircularRegion `实例从指定的(geotification)中。
4. 最后，你用Core Location注册监听`CLCircularRegion `实例

当用户删除地理围栏的时候，你同样需要停止监听

在`GeotificationsViewController.swift`中，添加如下方法:

```
func stopMonitoring(geotification: Geotification) {
  for region in locationManager.monitoredRegions {
    guard let circularRegion = region as? CLCircularRegion, circularRegion.identifier == geotification.identifier else { continue }
    locationManager.stopMonitoring(for: circularRegion)
  }
}
```

这个方法执行调用`locationManager `去停止监听`CLCircularRegion `通过`geotification `对象去判断

现在你已经完成了开始和停止的方法，当你不管什么时候添加和删除geotification,你将会调用到这两个方法

首先，在`GeotificationsViewController.swift`中找到`addGeotificationViewController(_:didAddCoordinate)`方法，这个方法是一个代理方法，在创建geotification的时候调用；

当创建一个新的新的`Geotification `通过 AddGeotificationsViewController，同时更新mapView和`geotifications `的集合，然后调用`saveAllGeotifications()`进行数据保存，最新的数据是被保存到`NSUserDefaults `中。

现在，替换为下面的代码:

```
func addGeotificationViewController(controller: AddGeotificationViewController, didAddCoordinate coordinate: CLLocationCoordinate2D, radius: Double, identifier: String, note: String, eventType: EventType) {
  controller.dismiss(animated: true, completion: nil)
  // 1
  let clampedRadius = min(radius, locationManager.maximumRegionMonitoringDistance)
  let geotification = Geotification(coordinate: coordinate, radius: clampedRadius, identifier: identifier, note: note, eventType: eventType)
  add(geotification: geotification)
  // 2
  startMonitoring(geotification: geotification)
  saveAllGeotifications()
}
```

1. 你调用了location manager的`maximumRegionMonitoringDistance `属性和半径值进行比较，如果半径值超过了这个属性值，则获取该属性值，反之则取半径值，这是重要的一点，因为任何大于最大值的半径将会引发监听程序失败
2. 用core location对新创建的geotification进行监听，通过调用` startMonitoringGeotification(_:)`方法，参数为`geofence `

通过这些代码，这个注册的App是有能力去进行监听，然而，有一个限制，作为地理围栏是和系统资源共享的，Core loaction要求每个设备最多只能注册20个地理围栏。

添加如下代码在`updateGeotificationsCount()`方法中:

```
func updateGeotificationsCount() {
  title = "Geotifications (\(geotifications.count))"
  navigationItem.rightBarButtonItem?.isEnabled = (geotifications.count < 20)  // Add this line
}
```

这行代码意义就是当数量达到限制数量值得时候，导航栏的`Add`按钮将会变成`不可用`状态

最后，让我们处理删除地理围栏的操作，这个函数是在`mapView(_:annotationView:calloutAccessoryControlTapped:)`这个代理方法中进行处理的，当用户点击每一个annotationView的左边的`delete`按钮的时候，将调用删除地理围栏的处理函数

添加停止监听的方法在`mapView(_:annotationView:calloutAccessoryControlTapped:)`代理方法调用的时候：

```
func mapView(_ mapView: MKMapView, annotationView view: MKAnnotationView, calloutAccessoryControlTapped control: UIControl) {
  // Delete geotification
  let geotification = view.annotation as! Geotification
  stopMonitoring(geotification: geotification)   // Add this statement
  removeGeotification(geotification)
  saveAllGeotifications()
}
```

添加声明去停止监听地理围栏通过传递`geotification `参数，在删除之前，请先改变`NSUserDefaults `中存储的值

通过这个方法，你的App可以停止监听地理围栏。


编译运行，你不会看到任何改变，但是这个App已经注册成了地理围栏的监听并且可以监听该区域，然而，它还不能响应任何的地理围栏的监听事件，不要着急-那就是你下一步需要做的事情.

![1](https://koenig-media.raywenderlich.com/uploads/2015/03/These_eyes.png)

## 响应地理围栏的事件

你将要实现一些当发生错误时的代理方法，当发生错误的时候这些代理将会被调用.

在`GeotificationsViewController.swift`中，添加CLLocationManagerDelegate的代理方法：

```
func locationManager(_ manager: CLLocationManager, monitoringDidFailFor region: CLRegion?, withError error: Error) {
  print("Monitoring failed for region with identifier: \(region!.identifier)")
}
 
func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
  print("Location Manager failed with the following error: \(error)")
}
```

这些代理方法仅仅是打印一些日志信息，当location manager发生错误的时候

下一步，打开` AppDelegate.swif`，在这里你将要添加一些代理用来处理和响应当设备进入或者离开事件

首先你需要导入`Corelocation`框架

```
import CoreLocation
```

确保在AppDelegate的顶端中有`CLLocationManager `的实例：

```
class AppDelegate: UIResponder, UIApplicationDelegate {
  var window: UIWindow?
 
  let locationManager = CLLocationManager() // Add this statement
  ...
}
```

替换`application(_:didFinishLaunchingWithOptions:)`中的实现:
```
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey : Any]? = nil) -> Bool {
  locationManager.delegate = self
  locationManager.requestAlwaysAuthorization()
  return true
}
```

你设置你的AppDelegate去接收地理围栏的事件，你可能好奇，“为什么我要用AppDelegate去替换ViewController呢?”

注册了一个地理围栏随时都在监听，包括当这个App停止运行的时候。假如这个设备是在App停止运行的时候去触发，ios将会自动进入后台运行模式，这使得AppDelegate是一个理想的地方去处理这个事件，这个时候ViewController可能不会被加载。

现在你可能还会好奇，“新创建的`CLLocationManager `实例是如何知道去监听地理围栏的呢?”

你的app注册的地理围栏是很容易去监听的，所以不会担心你的locaion Manager是什么地方初始化的。

现在你只需要实现代理方法去响应地位围栏的事件信息，在你做这些之前，你将要创建一个方法去处理地理围栏的事件。

添加下面的方法在AppDelegate.swift中

```
func handleEvent(forRegion region: CLRegion!) {
  print("Geofence triggered!")
}
```

在这个方法中，传递一个CLRegion参数，并且打印一个日志声明，稍后你将会实现这个事件处理。

下一步，在AppDelegate.swift的扩展中，添加下面的代理方法:

```
extension AppDelegate: CLLocationManagerDelegate {
 
  func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
    if region is CLCircularRegion {
      handleEvent(forRegion: region)
    }
  }
 
  func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion) {
    if region is CLCircularRegion {
      handleEvent(forRegion: region)
    }
  }
}
```
当你的设备进入`CLRegion`区域时，`locationManager(_:didEnterRegion:)`方法将会被调用，当你离开该区域时，`locationManager(_:didExitRegion:) `方法将会被调用

两个方法都会返回`CLRegion `，你需要去检查并且确保它是`CLCircularRegion `，因为返回有可能是`CLBeaconRegion `，假如你的App使用iBeacons来进行监听的。


假如你的region属于`CLCircularRegion `返回之内，你将会调用`handleRegionEvent(_:)`

现在你的App是能收到地理围栏的事件了，你要准备去测试它的准备性，如果这个不足以让你兴奋，因为在这个教程的第一次测试，你想要看到一些结果;

测试你的App最精确的方式是用真机，添加一些地理围栏并且进行走路或者驾驶汽车测试，然而，现在做这些有些不明智，因为你不能去验证打印的日志信息当用真机改变了地理位置之后，另外，在你提交一个大头针之前，它不能很好的保证App的工作。

幸运的是，这里有一个更容易的方式去做这个，你不用离开你舒适的家。

Xcode允许你包含一个WayPoint文件在你工程中，你能用Monique去测试地点位置。很幸运吧，你在你工程中导入这个文件即可。

打开`TestLocations.gpx`这个文件，检查下这个内容：

```
<?xml version="1.0"?>
<gpx version="1.1" creator="Xcode">
  <wpt lat="37.422" lon="-122.084058">
    <name>Google</name>
  </wpt>
  <wpt lat="37.3270145" lon="-122.0310273">
    <name>Apple</name>
  </wpt>
</gpx>
```


这个GPX文件一个XML格式的，包含两个waypoints:`Google’s Googleplex in Mountain View and Apple’s Headquarters in Cupertino.`

用模拟器运行工程，当App加载这Main View Controller时，回到Xcode,在Debug bar中选中Locationt图标，选择`TestLocations `

![2](https://koenig-media.raywenderlich.com/uploads/2015/03/Screen-Shot-2015-02-02-at-9.04.37-pm.png)

回到App中，点击导航栏上的左上角的Zoom按钮，定位当前位置，一旦你放大这个区域，你将会看到定位点在Google 像素点和 Apple像素点 来回移动。

添加两个地理围栏去测试这个App,Apple坐标和Google坐标，（如果之前你添加过其他的地理围栏，请先删除之前的，再开始）

为了测试这些地点，添加具体细节如下:

* Google:Radius: 1000m, Message: “Say Bye to Google!”, Notify on Exit
* Apple:Radius: 1000m, Message: “Say Hi to Apple!”, Notify on Entry

![1](https://koenig-media.raywenderlich.com/uploads/2016/06/Geo2Fences-281x500.png)

一旦你添加了地理围栏信息，你将会看到每次当坐标点进入或者离开地理围栏的时候，控制台都会打印日志信息，如果你按下home键或者锁屏让App进入到后台，你将会看到日志信息每次都会打印，现在你可以很显然的验证之前的判断

![1](https://koenig-media.raywenderlich.com/uploads/2015/03/GeofenceTriggered.png)

## 用通知来响应地理围栏事件

你已经有了很大的进步，当你的设备通过这个地理围栏的时候，你可以发送一个通知提示用户。准备好做下这个操作

为了获得通知消息，你可以通过触发`CLCircularRegion `来获得，你需要获取一个地理围栏信息从`NSUserDefaults `存储的数据中，在已经注册的地理围栏中，你可以用这个唯一的`identifier`去找到正确的`CLCircularRegion `。

在AppDelegate.swift中，添加下面的帮助方法：

```
func note(fromRegionIdentifier identifier: String) -> String? {
  let savedItems = UserDefaults.standard.array(forKey: PreferencesKeys.savedItems) as? [NSData]
  let geotifications = savedItems?.map { NSKeyedUnarchiver.unarchiveObject(with: $0 as Data) as? Geotification }
  let index = geotifications?.index { $0?.identifier == identifier }
  return index != nil ? geotifications?[index!]?.note : nil
}
```

这个方法会从持久化的内存中找到地理围栏的消息，仅仅靠`identifier `，就会返回地理围栏的消息


在` application(_:didFinishLaunchingWithOptions:)`添加如下方法，去注册一个通知:

```
application.registerUserNotificationSettings(UIUserNotificationSettings(types: [.sound, .alert, .badge], categories: nil))
UIApplication.shared.cancelAllLocalNotifications()
```

你添加的提示权限是为了确保这个App能否发送通知

现在，替换`handleRegionEvent(_:)`的内容:

```
func handleEvent(forRegion region: CLRegion!) {
  // Show an alert if application is active
  if UIApplication.shared.applicationState == .active {
    guard let message = note(fromRegionIdentifier: region.identifier) else { return }
    window?.rootViewController?.showAlert(withTitle: nil, message: message)
  } else {
    // Otherwise present a local notification
    let notification = UILocalNotification()
    notification.alertBody = note(fromRegionIdentifier: region.identifier)
    notification.soundName = "Default"
    UIApplication.shared.presentLocalNotificationNow(notification)
  }
}
```
当这个App激活的时候，出现主界面的时候，将会弹出一个alert提示信息

编译运行工程，当坐标点经过你添加的地理围栏的时候，你的地理围栏事件将会被触发，将会受到一个提示信息，并且展示出来:

![1](https://koenig-media.raywenderlich.com/uploads/2016/06/GeoSayByeToGoogle-281x500.png)


按一下home按钮，使得App进入后台模式，你依然能够通过地理围栏的事件信号定期收到通知信息

![1](https://koenig-media.raywenderlich.com/uploads/2016/09/IMG_2239-281x500.png)


 现在，你已经有完整的功能，定位提示功能在你的App中，你可以添加其他的大头针，然后去到你添加坐标的地方测试~
 
## 最终工程

最后的工程你可以在这里[下载](https://koenig-media.raywenderlich.com/uploads/2016/09/Geotify-Final-1.zip)







