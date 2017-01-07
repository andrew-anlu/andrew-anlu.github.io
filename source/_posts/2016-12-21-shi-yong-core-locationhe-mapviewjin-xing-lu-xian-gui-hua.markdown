---
layout: post
title: "使用Core Location和MapView进行路线规划"
date: 2016-12-21 16:04:20 +0800
comments: true
categories: swift
---


## 开始

[下载开始工程](https://koenig-media.raywenderlich.com/uploads/2015/11/ProcrastinatorsRevenge-starter.zip)

并且打开`ProcrastinatorsRevenge.xcodeproj `,编译运行，界面效果如下:

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/starter_screenshots1.png)
<!--more-->

第一个屏幕是输入文本框，设置开始和终止地点，点击`Route it`,App将会跳转到第二个页面，开始规划路线图。


## 使用MapKit和Coreloaction

官方文档中对Core Loaction的描述是这样的:"Core location框架让你决定当前的坐标或者或者当前的方位通过使用设备"，你将要用Core Location的特性去填充用户的开始点，Core Location能翻译坐标的经纬度为用户能够看得懂的地址信息。在MapView中，使用MapItem类的`CLGeocoder `类将会完成这个教程的第一部分。



在这个教程的第二部分，你将要从`CLGeocoder `返回的`CLPlacemark `中转为一个`MKPlacemark `，然后把`MKPlacemark `转换为一个`MKMapItem `。你将要使用`MKMapItems `去运行一个`MKDirectionsRequest `，这个最终将会返回一个`MKRoute `对象。

关系如下->`CLGeocoder > CLPlacemark > MKPlacemark > MKMapItem > MKDirectionsRequest > MKRoute.`

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/ef1905fa6eccb7e1cd85141d065d49f8.jpg-320x320.gif)

## 通过CoreLocation获取当前的坐标信息

在`ViewController.swift`中，添加如下代码，替换`viewDidLoad `已经存在的代码

```
// 1
let locationManager = CLLocationManager()
 
override func viewDidLoad() {
  super.viewDidLoad()
  originalTopMargin = topMarginConstraint.constant
  // 2
  locationManager.delegate = self
  locationManager.requestWhenInUseAuthorization()
  // 3
  if CLLocationManager.locationServicesEnabled() {
    locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
    locationManager.requestLocation()
  }
}
```

让我们说说每一步都做了什么:

1. 你定义了一个全局常量`locationManager `
2. 在`ViewDidLoad`中，设置location manager的代理，当用户打开App的时候，询问用户是否允许访问用户地理位置的权限，这个弹出框第一次将会显示，当用户做出响应之后，这个框就不会再出现了。
3. 一旦用户定位服务开启，设置这个`CLLocationManager’s `的定位精度信息，请求当前坐标


编译运行你的程序，你是否弹出了提示框让你授权地理定位信息呢？

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/reaction_faces___confused_by_awesomesauceuk-d4od42z.png)

这是因为你还有一件事情需要完成，那就是权限问题，你需要提供原因为你的请求。
打开`Supoorting Files>info.plist`，按如下步骤进行设置:

1. 添加`NSLocationWhenInUseUsageDescription `作为`Key`在 Information Property List中
2. 让这个`Type`为`String`类型
3. 设置这个`Value`值去向用户展示，解释为什么你需要访问他们的地理信息：“请运行我们访问你的当前坐标信息，这样我们就能自动填充你的开始和结束地点了”


>`注意:`
>*.requestWhenInUseAuthorization()* 让这个app访问当前的用户地理信息，当这个App是在使用的时候
>
>*.requestAlwaysAuthorization() * 当这个app不管是在前台还是后台的时候.这个app都能访问用户的地理信息，
>
>
>


![1](https://koenig-media.raywenderlich.com/uploads/2015/07/plist.png)

编译运行你的app,这次提示框将会显示:

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/LocationAuthorization.png)

点击`Allow`,这个location Manager现在将会访问到你的地理坐标

下一步，你将要创建`CLGeocoder `去解码当前的坐标信息。解码过程就是把获取到的地理坐标，比如经纬度，转换成我们能看的懂得地理位置信息，比如长江道第三大街33号

在`ViewConroller.swift`的底部，添加如下代码，`locationManager(_:didUpdateLocations:locations:):`

```
CLGeocoder().reverseGeocodeLocation(locations.last!,
  completionHandler: {(placemarks:[CLPlacemark]?, error:NSError?) -> Void in
  if let placemarks = placemarks {
    let placemark = placemarks[0]
  }
})
```


`reverseGeocodeLocation(_:completionHandler:)`将会返回一组`placemarks `，对于多数的`geocoding `结果来说，这个数组仅仅是包含一个元素;很少的情况，一个单独的地点能返回多个地理位置附近的信息。在这个例子中，我们获取第一个placemark,即：placemarks[0],将会满足需求。

你可以可以停止更新坐标信息，当你发现了一个适当的placemark的时候。


现在你通过用户的地理位置已经找到对应地理坐标的`CLPlacemark `，你需要去联想其它跟他相关的地理数据信息，通过用户输入的文本框中的位置。为了实现这个，当用户输入一个单独的值的时候，通过swift的元组结构值去匹配多个值。

在`ViewDidLoad`中，添加下面的全局变量去匹配每个`UITextField `响应的`MKMapItem`：

```
var locationTuples: [(textField: UITextField!, mapItem: MKMapItem?)]!
```


对用户的坐标信息，你将要存储`MKMapItems `而不是`CLPlacemarks `，这个类型你最终将会用在初始化`MKDirectionsRequest `上，用来计算路径规划。

在`ViewDidLoad`中，添加如下:

```
locationTuples = [(sourceField, nil), (destinationField1, nil), (destinationField2, nil)]
```

这里，你定义了一个数组，里面包含了元组,每个元组都包含了一个文本框和一个为空的 MKMapItem的值，这个值最终会被和文本框绑定到一起。

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/happy-thumbs-up.png)

在`locationManager(_:didUpdateLocations:location:)`方法中，添加如下代码片段，在`reverseGeocodeLocation(_:completionHandler:)`的完成回调方法中:

```
self.locationTuples[0].mapItem = MKMapItem(placemark:
  MKPlacemark(coordinate: placemark.location!.coordinate,
  addressDictionary: placemark.addressDictionary as! [String:AnyObject]?))
```

添加了一个`MKMapItem `代表用户当前的地理位置作为`locationTuples `的第一个元组对象。

下一步，在ViewController中添加下面的函数 ，去把location data中的地理信息提取出来

```
func formatAddressFromPlacemark(placemark: CLPlacemark) -> String {
  return (placemark.addressDictionary!["FormattedAddressLines"] as! 
    [String]).joinWithSeparator(", ")
}
```

`formatAddressFromPlacemark(_:)`从`CLPlacemark's `地址字典中通过`FormattedAddressLines `key提取一组地址信息，然后每两个元素之间用逗号连接起来组成一个新的字符串。





滑动到`locationManager(_:didUpdateLocations:locations:)`方法，在self.locationTuples[0].mapItem初始化后添加如下代码:

```
self.sourceField.text = self.formatAddressFromPlacemark(placemark)
```

这将设置UItextField为一个新地址。

```
self.enterButtonArray.filter{$0.tag == 1}.first!.selected = true
```

在开始工程中，按钮的选择文本是提前设置好的，按钮的tag是按照顺序设置的，每个`Enter`按钮都和页面`enterButtonArray `建立了连接关系，上面的代码找到 tag=1的 `Enter`按钮，并且每个UItextField也是按照顺序设置的tag，和按钮是一一对应的。

所以按钮的状态为选中时的文本会变为`✓`。

编译运行你的APP，假如当前坐标是:Apple HQ,你的文本框中的文本将会是:`Apple Inc., 2 Infinite Loop, Cupertino, CA 95014-2083, United States`

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/Apple_source.png)


在模拟器中，修改当前的坐标 `select Debug > Location > Custom location…:`
![1](https://koenig-media.raywenderlich.com/uploads/2015/07/Menu_bar-700x374.png)

输入坐标信息:

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/CustomLocation-e1437159760381.png)

`Latitude: 29.049186, Longitude: -95.45384`

编译运行程序，这个文本框中将会显示:`05 Any Way St, Lake Jackson, TX, 77566-4198, United States
`

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/Any_Way_source.png)

下一步，你需要用户正确的输入地址，然后通过文本框中的值获取关联的地理位置信息，通过创建`MKMapItems `来获取。

## 用户调用CoreLoaction过程

在`ViewController.swift`中，更新`addressEntered(_:)`代码:

```
@IBAction func addressEntered(sender: UIButton) {
  view.endEditing(true)
  // 1
  let currentTextField = locationTuples[sender.tag-1].textField
  // 2
  CLGeocoder().geocodeAddressString(currentTextField.text!,
    completionHandler: {(placemarks: [CLPlacemark]?, error: NSError?) -> Void in
    if let placemarks = placemarks {
 
    } else {
 
    }
  })
}
```
你添加的代码做的工作如下:

1. 在interface builder中，每个`Enter`按钮都有一个tag,从上到下依次为1,2,3.你可以通过sender.tag来找到是那个text field.(UItextField的tag和Enter按钮的tag是一一对应的)
2. 通过` CLGeocoder's geocodeAddressString(_:completionHandler:).`去解码地理坐标

不像`reverseGeocodeLocation(_:completionHandler:), geocodeAddressString(_:completionHandler:) `，经常会返回多个`CLPlacemark`信息，在文本中输入的值经常不是仅仅匹配一个值。幸运的是，我们创建一个Tableview来显示多个数据信息，当用户选择了一项之后我们就会返回一个`CLPlacemarks`

看一下`AddressTableView.swift.`这个类，你应该很清楚`tableView(_:numberOfRowsInSection:) `和`tableView(_:cellForRowAtIndexPath:),`代理方法的作用，你将要使用在顶部定义的全局变量`address`数组去填充这个tableview.

在`ViewController `中添加如下的函数:

```
func showAddressTable(addresses: [String]) {
  let addressTableView = AddressTableView(frame: UIScreen.mainScreen().bounds,
    style: UITableViewStyle.Plain)
  addressTableView.addresses = addresses
  addressTableView.delegate = addressTableView
  addressTableView.dataSource = addressTableView
  view.addSubview(addressTableView)
}
```

通过方法`geocodeAddressString(_:completionHandler:).`返回一组`CLPlacemarks `数据。这里你创建一个AddressTable，并且设置了数据源`addresses`数组，数组中包含的元素就是`CLPlacemarks `


回到`addressEntered(_:)`方法中，在`if let placemarks = placemarks`回调函数`geocodeAddressString(_:completionHandler:)‘s `中，添加如下代码:

```
var addresses = [String]()
for placemark in placemarks {
  addresses.append(self.formatAddressFromPlacemark(placemark))
}
self.showAddressTable(addresses)
```

你看到了属性`placemarks`，遍历访问并且把地址追加到Addresses数组中。最后把`addresses `作为参数传递给`self.showAddressTable `

当在第一屏幕的文本框中输入地址，并且点击`Enter`按钮，就会跳转到一个页面，让你选择具体是那个地址:

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/OysterCreekTable.png)


当你选择了其中了一个地址之后，会发生什么呢？这个table将会消失。好了，让我去实现它

当你选择另一个地址之后，你想要自动的设置选择的文本去填充第一屏幕中的文本框中的值，更新查询到的locations数组，并且设置相关的`MKMapItem`,并且设置`Enter`按钮的选中状态。

更新`showAddressTable(_:)`方法在`ViewController.swift`中:

```
func showAddressTable(addresses: [String], textField: UITextField,
  placemarks: [CLPlacemark], sender: UIButton) {
 
  let addressTableView = AddressTableView(frame: UIScreen.mainScreen().bounds, style: UITableViewStyle.Plain)
  addressTableView.addresses = addresses
  addressTableView.currentTextField = textField
  addressTableView.placemarkArray = placemarks
  addressTableView.mainViewController = self
  addressTableView.sender = sender
  addressTableView.delegate = addressTableView
  addressTableView.dataSource = addressTableView
  view.addSubview(addressTableView)
}
```

这里，你通过当前的文本框字段，数组placemarks,和当前的`ViewController.swift `的实例 创建一个AddressTableView。


在`geocodeAddressString(_:completionHandler:)`方法中，更新`showAddressTable(_:) `的参数:

```
self.showAddressTable(addresses, textField: currentTextField,
    placemarks: placemarks, sender: sender)
```

在查询不到地址的情况下，应该马上弹出提示框:

```

self.showAlert("Address not found.")
```

如果`geocodeAddressString(_:completionHandler:)`没有返回任何placemarks,你应该显示一个error:

下一步，在表格的点击方法`tableView(_:didSelectRowAtIndexPath:)`中，加入代码:

```
// 1
if addresses.count > indexPath.row {
  // 2
  currentTextField.text = addresses[indexPath.row]
  // 3
  let mapItem = MKMapItem(placemark:
    MKPlacemark(coordinate: placemarkArray[indexPath.row].location!.coordinate,
    addressDictionary: placemarkArray[indexPath.row].addressDictionary
    as! [String:AnyObject]?))
  mainViewController.locationTuples[currentTextField.tag-1].mapItem = mapItem
  // 4
  sender.selected = true
}
```

下面解释代码的意思:

1. 当addresses的数量是大于当前行的索引的时候,表格的最后一行将会显示:`None of the above`
2. 更新当前的文本框中的值为选择的地址
3. 使用placemark去创建创建`MKMapItem `，placemark是通过获取当前行创建的。根据当前文本框的tag获取`mainViewController's locationTuples`中的mapitem,并且设置它
4. 设置当前`Enter`为选中状态


编译运行，在`Stop#1`中输入地址，点击`Enter`，然后选择正确的地址，这个文本框中的值，地址元组(location tuple arra)，还有`Enter`按钮将会自动更新:
![1](https://koenig-media.raywenderlich.com/uploads/2015/07/Two_entries.png)

*Addresses updated. (Checkmarks et al.)*


在`ViewController.swift`中你还有一部分工作需要完成。

更新代码在`textField(_:shouldChangeCharactersInRange:replacementString:):`

```
func textField(textField: UITextField,
  shouldChangeCharactersInRange range: NSRange,
  replacementString string: String) -> Bool {
 
  enterButtonArray.filter{$0.tag == textField.tag}.first!.selected = false
  locationTuples[textField.tag-1].mapItem = nil
  return true
}
```

当用户编辑了文本框，你将要设置MKMapItem失效，因为MKMapItem是不再需要，这样用户就可以重新选择正确的地址。

下一步，更新` swapFields(_:)`方法:

```

swap(&destinationField1.text, &destinationField2.text)
    swap(&locationTuples[1].mapItem, &locationTuples[2].mapItem)
    swap(&self.enterButtonArray.filter{$0.tag == 2}.first!.isSelected, &self.enterButtonArray.filter{$0.tag == 3}.first!.isSelected)
```

当用户点击`↑↓`，你需要交换两个文本框的值，还有在数组`locationTuples `对应的MKMapItems

在`ViewController`类中，找到`getDirections(_:)`方法，覆盖`shouldPerformSegueWithIdentifier(_:sender:)`：

```
override func shouldPerformSegueWithIdentifier(identifier: String, sender: AnyObject?) -> Bool {
  if locationTuples[0].mapItem == nil ||
    (locationTuples[1].mapItem == nil && locationTuples[2].mapItem == nil) {
    showAlert("Please enter a valid starting point and at least one destination.")
    return false
  } else {
    return true
  }
}
```

在`ViewDidload:`中对`locationsArray `进行设置，它是一个只读的属性，当你访问它的时候，它会从`locationTuples `中过滤`mapItem `不为nil的元素.


```
var locationsArray: [(textField: UITextField!, mapItem: MKMapItem?)] {
  var filtered = locationTuples.filter({ $0.mapItem != nil })
  filtered += [filtered.first!]
  return filtered
}
```

`filtered += [filtered.first!]`拷贝元组中的第一个元素的值作为这个数组的最后一个元素。

在`prepareForSegue(_:sender:):`中:

```
override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
  var directionsViewController = segue.destinationViewController as! DirectionsViewController
  directionsViewController.locationArray = locationsArray
}
```


这个传递的参数`locationsArray `将会在下一个View controller中用到

现在切换到`DirectionsViewController.swift`，去开始路径规划吧.

## 在MapKit中进行路径规划

现在，你已经知道了addresses,你将要创建一个`MKDirections `对象，调用`calculateDirectionsWithCompletionHandler(_:) `设置开始坐标和结束坐标就可以开始路径规划了.

在DirectionsViewController中添加如下代码:

```
func calculateSegmentDirections(index: Int) {
  // 1
  let request: MKDirectionsRequest = MKDirectionsRequest()
  request.source = locationArray[index].mapItem
  request.destination = locationArray[index+1].mapItem
  // 2
  request.requestsAlternateRoutes = true
  // 3
  request.transportType = .Automobile
  // 4
  let directions = MKDirections(request: request)
  directions.calculateDirectionsWithCompletionHandler ({
    (response: MKDirectionsResponse?, error: NSError?) in
    if let routeResponse = response?.routes {
 
    } else if let _ = error {
 
    }
  })
}
```

上面的代码做的工作如下:

1. 创建一个`MKDirectionsRequest `，通过索引获取`locationArray `对应的mapItem,设置MKDirectionsRequest的开始坐标为这个mapitem,设置索引的下一个(index+1)为这个请求的目的地坐标
2. 设置`requestsAlternateRoutes `为true,请求从源目标到目的地所有可能的路径
3. 设置交通方式为 汽车驾驶,其它几个可能的方式为:步行，所有,公交车等
4. 通过`MKDirectionsRequest `初始化`MKDirections `，然后调用`calculateDirectionsWithCompletionHandler `获取到一个`MKDirectionsResponse `，这会包含一组`MKRoutes `数据

如果`calculateDirectionsWithCompletionHandler(_:)`没有返回任何routes,而是返回一个错误，那么`else if let _ = error`代码将会执行，添加这个代码:

```
let alert = UIAlertController(title: nil,
  message: "Directions not available.", preferredStyle: .Alert)
let okButton = UIAlertAction(title: "OK",
  style: .Cancel) { (alert) -> Void in
  self.navigationController?.popViewControllerAnimated(true)
}
alert.addAction(okButton)
self.presentViewController(alert, animated: true,
  completion: nil)
```


假设`MKRoutes `是找到了，在第一个`if let `声明中`calculateDirectionsWithCompletionHandler(_:) `将会执行，添加如下代码:

```

let quickestRouteForSegment: MKRoute =
  routeResponse.sort({$0.expectedTravelTime <
  $1.expectedTravelTime})[0]
```

这里你按照 到达的时间 进行排序，用时最短的时间将会排在第一个，这对你的路线规划来说是有好处的。
![2](https://koenig-media.raywenderlich.com/uploads/2015/07/troll-troll-dad-dance-jump.png)

但是，你仍然需要计算多个路径在每两个地点之间。

首先，更新`calculateSegmentDirections(_:)`参数:

```
func calculateSegmentDirections(index: Int,
  time: NSTimeInterval, routes: [MKRoute]) {
```

`calculateSegmentDirections(_:time:routes:)`现在接受一个数组和一个 NSTimeInterval 参数

```

// 1
var timeVar = time
var routeVar = routes
//2
routesVar.append(quickestRouteForSegment)
// 3
timeVar += quickestRouteForSegment.expectedTravelTime
// 4
if index+2 < self.locationArray.count {
  self.calculateSegmentDirections(index+1, time: timeVar, routes: routesVar)
} else {
 
}
```

1. 创建两个变量，一个是timeVar,一个是routeVar
2. 为当前的分段路线routesVar添加最快的路径规划
3. 在timeVar中添加路径规划的预计到达时间
4. 当你当前的索引加上最后两个值后没有超过`location array`数量值，递归调用自身方法`calculateSegmentDirections(_:time:routes:)`，传递索引+1,当前的time和路径values.

现在回到`viewDidLoad `中，添加如下代码：

```
addActivityIndicator()
calculateSegmentDirections(0, time: 0, routes: [])
```

这个代码添加一个转子activity indicator,当路径规划开始计算的时候，然后调用`calculateSegmentDirections(_:time:routes:)`去计算路径，从locationArray第一个索引开始，初始化时间为0,初始化一个空的路径数组

然后回到`calculateDirectionsWithCompletionHandler(_:)`中，在`else`的回调函数判断中，在` if index+2 < self.locationArray.count`的下面添加:

```
self.hideActivityIndicator()
```

当计算完所有的路径规划后，隐藏转子

## 在MKMapIVew中添加MKRoutes
为了去规划每一个`MKMapView `在MKMapView中，你需要在`DirectionsViewController `添加如下方法:

```
func plotPolyline(route: MKRoute) {
  // 1
  mapView.addOverlay(route.polyline)
  // 2
  if mapView.overlays.count == 1 {
    mapView.setVisibleMapRect(route.polyline.boundingMapRect,
      edgePadding: UIEdgeInsetsMake(10.0, 10.0, 10.0, 10.0),
      animated: false)
  }
  // 3
  else {
    let polylineBoundingRect =  MKMapRectUnion(mapView.visibleMapRect,
      route.polyline.boundingMapRect)
    mapView.setVisibleMapRect(polylineBoundingRect,
      edgePadding: UIEdgeInsetsMake(10.0, 10.0, 10.0, 10.0),
      animated: false)
  }
}
```

`plotPolyline(_:)`做的工作如下:

1. 添加`MKRoute`到地图上，作为一个覆盖层
2. 如果规划的路径仅仅是一个覆盖层，设置地图的可见区域足够大足以填充整个覆盖层，外边缘留出10个点的间距
3. 如果规划的路径不是一个，设置地图的可见区域为新的和旧的的区域的联合，外边缘留出10个点的间距

下一步，更新`mapView(_:rendererForOverlay:)`方法

```
func mapView(mapView: MKMapView,
  rendererForOverlay overlay: MKOverlay) -> MKOverlayRenderer! {
 
  let polylineRenderer = MKPolylineRenderer(overlay: overlay)
  if (overlay is MKPolyline) {
    if mapView.overlays.count == 1 {
      polylineRenderer.strokeColor =
        UIColor.blueColor().colorWithAlphaComponent(0.75)
    } else if mapView.overlays.count == 2 {
      polylineRenderer.strokeColor =
        UIColor.greenColor().colorWithAlphaComponent(0.75)
    } else if mapView.overlays.count == 3 {
      polylineRenderer.strokeColor =
        UIColor.redColor().colorWithAlphaComponent(0.75)
    }
    polylineRenderer.lineWidth = 5
  }
  return polylineRenderer
}
```

给每一个路径路段设置不同的颜色

在`calculateSegmentDirections(_:time:routes:)`添加:

```
func showRoute(routes: [MKRoute]) {
  for i in 0..<routes.count {
    plotPolyline(routes[i])
  }
}
```

这个函数循环所有的`MKRoute `，添加覆盖层到地图上。

在方法`calculateDirectionsWithCompletionHandler(_:)`中，调用`showRoute(_:)`方法。在`else`block中你调用的`self.hideActivityIndicator():`的代码上面调用

```
self.showRoute(routesVar)
```

编译运行，输入地址然后点击`Route it`,路线规划将要出现在地图上:

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/Map_Route.png)

下一步，你要打印路线的每一步路径信息到`DirectionsTable`上

## 打印MKRoute的路径信息

`DirectionsTable.swift `包含一个全局的数组`directionsArray `,是一个包含元组的数组(string,string,MKRoute).这两个字符串是开始地址和结束地址，最后一个MKRoute，是路径的信息。


滑动页面到`UITableViewDataSource `的扩展方法中，在代理方法`numberOfSectionsInTableView(_:)`设置返回的数量，这个表格将要包含一个section为存储在`directionsArray `中每一个route。

这样你的`DirectionsTable `的每一个section就会为route呈现一个不同的分段.

代理方法` tableView(_:numberOfRowsInSection:)`将会返回`MKRouteSteps `的数量，比如:`directionsArray[section].route.`

从开始地点到目的地一共有多少步.

在`tableView(_:cellForRowAtIndexPath:)`中，添加如下代码:

```
// 1
let steps = directionsArray[indexPath.section].route.steps
// 2
let step = steps[indexPath.row]
// 3
let instructions = step.instructions
// 4
let distance = step.distance.miles()
// 5
cell.textLabel?.text = "\(indexPath.row+1). \(instructions) - \(distance) miles"
```

在每一行中，你打印`MKRouteStep `的说明信息和距离等:

1. 通过当前的section的值，得到`MKRoute `,然后通过`MKRoute`获取`MKRouteSteps `的数组
2. 从steps的数组中，访问`MKRouteStep `的对象通过当前行。
3. 获取step的说明信息
4. 获取step的距离信息，在`CLLocationDistance `的扩展方法中，定义了一个miles()方法，通过这个方法把距离转为 米 单位 。
5. 设置label显示出每一步的说明信息和距离

现在，你可以使用`UITableViewDelegate `的扩展方法去显示开始地点和结束地点的信息。可以在每一个section中的header中显示开始地址，在footer中显示结束地点。

在`tableView(_:viewForHeaderInSection:)`添加如下代码:

```
label.text = "SEGMENT #\(section+1)\n\nStarting point: \(directionsArray[section].startingAddress)\n"

```
这个头部view包含了开始地址:

在`tableView(_:viewForFooterInSection:)`中添加如下代码:

```
// 1
let route = directionsArray[section].route
// 2
let time = route.expectedTravelTime.formatted()
// 3
let miles = route.distance.miles()
//4
label.text = "Ending point: \(directionsArray[section].endingAddress)\n\nDistance: \(miles) miles\n\nExpected Travel Time: \(time)"
```



1. 得到当前的section route
2. 从NSTimeInterval的扩展中，通过`formatted()`格式化`expectedTravelTime `,`formatted()`方法通过`NSDateComponentsFormatter `吧`NSTimeInterval `转为*时分秒*的格式
3. 通过`CLLocationDistance `格式化距离
4. 让表格的label显示结束地址，距离和期望到达时间信息

现在回到`DirectionsViewController `中，添加如下方法:

```
func displayDirections(directionsArray: [(startingAddress: String, 
  endingAddress: String, route: MKRoute)]) {
  directionsTableView.directionsArray = directionsArray
  directionsTableView.delegate = directionsTableView
  directionsTableView.dataSource = directionsTableView
  directionsTableView.reloadData()
}
```

更新`showRoute(_:):`方法:

```
 func showRoute(routes: [MKRoute]) {
  var directionsArray = [(startingAddress: String, endingAddress: String, route: MKRoute)]()
  for i in 0..<routes.count {
    plotPolyline(routes[i])
   let item = ((locationArray[i].textField?.text)!,
                        (locationArray[i+1].textField?.text)!, routes[i]) as TupleRouteItem
 //(startingAddress: String, endingAddress: String, route: MKRoute)
    directionsArray.append(item)
  }
  displayDirections(directionsArray)
}
```

对于每一个路径规划，你都添加了开始地址和结束地址，然后添加`MKRoute `到directionsArray数组中

下一步，你需要更新`totalTimeLabel `去显示总的期望达到时间。总的时间需要通过可变参数在`calculateSegmentDirections(_:time:routes:).`方法中计算出来。

所以更新`showRoute(_:) `方法，添加一个`NSTimeInterval `参数

```
func showRoute(routes: [MKRoute], time: NSTimeInterval) {
```

在`DirectionsViewController `类中添加如下方法

```
func printTimeToLabel(time: NSTimeInterval) {
  var timeString = time.formatted()
  totalTimeLabel.text = "Total Time: \(timeString)"
}
```

该方法在totalTimeLabel中显示出来总共时间

最后在函数的末尾，调用`printTimeToLabel(_:)`

```
printTimeToLabel(time)
```

编译运行:

![1](https://koenig-media.raywenderlich.com/uploads/2015/07/Route_2.png)


## 下载工程

[最终工程]()

