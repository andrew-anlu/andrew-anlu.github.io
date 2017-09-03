---
layout: post
title: "MapKit教程:起步"
date: 2016-12-15 15:02:58 +0800
comments: true
categories: swift
---

[MapKit](https://developer.apple.com/maps/)在ios开发中真的是很棒的API,它能很容易的展示地图和位置坐标，定位当前位置，可以路线规划甚至可以自定义覆盖物在地图上面

<!--more-->

在这个教程中，你将要创建一个app，在指定的地点能够放大缩小，并且可以设置大头针，支持点击操作，显示详情信息。

根据路线规划可以选择驾驶，步行等方式去往目的地。你的App将会解析json数据从服务器。

在这个过程中，你将会学会如何去添加一个MapKit在你的app中，放大或者缩小


## Getting Started

在Xcode中新建一个工程，命名为: HonoluluArtMapDemo;

然后打开ViewController中，在顶部引入 地图框架
`import MapKit`


然后手动创建一个mapView,代码如下:

```
 func initMap() -> Void {
        mapView = MKMapView(frame: CGRect(x: 0, y: 0, width: screenWidth, height: screenHeight))
        self.view.addSubview(mapView)
    }

```

## 设置可见区域

在顶部设置一个可见区域
```
// set initial location in Honolulu
let initialLocation = CLLocation(latitude: 21.282778, longitude: -157.829444)
```

你将要使用这个坐标去设置地图在地点Honolulu的位置。


当你尝试去告诉地图去显示什么的时候，你不能仅仅提供一个经度和纬度，对于定位中心点这个已经足够了，但是你还需要制定一个范围 region 去显示半径。

增加一个辅助方法:

```
let regionRadius: CLLocationDistance = 1000
func centerMapOnLocation(location: CLLocation) {
  let coordinateRegion = MKCoordinateRegionMakeWithDistance(location.coordinate, 
    regionRadius * 2.0, regionRadius * 2.0)
  mapView.setRegion(coordinateRegion, animated: true)
}
```

在`ViewDidload`中，添加如下代码：

```
initMap()
centerMapOnLocation(initialLocation)
```

### 运行测试
现在运行你的app,应该能看到一个地图，并且坐标是在 `Honolulu `

![1](https://koenig-media.raywenderlich.com/uploads/2015/03/run2.png)


## 获取Artworks Data

[HonoluluArtResources.zip](https://koenig-media.raywenderlich.com/uploads/2015/03/HonoluluArtResources.zip)是服务器上的json文件，下载解压后有两个文件：Public.json和JSON.swift.你将会用JSON.swift去解析Public.json

把这两个文件拖拽到你的工程中，确保`Destination: Copy items if needed`并且`Add to targets: HonoluluArt `是选择状态。



## 新建一个Artwork.swift 模型类




```
import UIKit
import MapKit

class Artwork: NSObject ,MKAnnotation {
    public var title: String?
    
    let locationName: String
    let discipline: String
    let coordinate: CLLocationCoordinate2D
    
    init(title: String, locationName: String, discipline: String, coordinate: CLLocationCoordinate2D) {
        self.title = title
        self.locationName = locationName
        self.discipline = discipline
        self.coordinate = coordinate
        
        super.init()
    }
    
    public var subtitle: String? {
        return locationName
    }
}

```

在这个模型类中，定义了标题和子标题，坐标和详细描述，并且实现了MKAnnotation协议。该协议中有两个属性必须实现:title 和 subtitle。

OK，这些属性title,locationName和coordinate属性将要被用于MKAnnotation上，但是`discipline `这个属性是做什么的呢？在这个教程中，你将会找到答案

接下来，在`ViewController`的 `ViewDiDload`方法中，加入如下代码:

```
// show artwork on map
let artwork = Artwork(title: "King David Kalakaua",
  locationName: "Waikiki Gateway Park", 
  discipline: "Sculpture",
  coordinate: CLLocationCoordinate2D(latitude: 21.283921, longitude: -157.831661))
 
mapView.addAnnotation(artwork)
```

创建了一个Artwork的对象，作为一个annotation加入到mapView 中。这个mapView有 addAnnotaion方法。

OK，接下来你需要展示这个annotaion在map view中，为了实现这个，你必须实现MKMapViewDelegate协议 `MKPinAnnotationView `该协议负责显示大头针annotation的


代码如下：

```

import MapKit
 
extension ViewController: MKMapViewDelegate {
 
  // 1
  func mapView(mapView: MKMapView!, viewForAnnotation annotation: MKAnnotation!) -> MKAnnotationView! {
    if let annotation = annotation as? Artwork {
      let identifier = "pin"
      var view: MKPinAnnotationView
      if let dequeuedView = mapView.dequeueReusableAnnotationViewWithIdentifier(identifier)
        as? MKPinAnnotationView { // 2
        dequeuedView.annotation = annotation
        view = dequeuedView
      } else {
        // 3
        view = MKPinAnnotationView(annotation: annotation, reuseIdentifier: identifier)
        view.canShowCallout = true
        view.calloutOffset = CGPoint(x: -5, y: 5)
        view.rightCalloutAccessoryView = UIButton.buttonWithType(.DetailDisclosure) as! UIView
      }
      return view
    }
    return nil
  }
}
```


这些代码并不复杂

1. `mapView(_:viewForAnnotation:) `是一个方法，每一个annotation都会调用去添加到mapView中，就像tableview中的`(_:cellForRowAtIndexPath:)`，每次都会返回一个View为每一个annotation
2. 和`tableview`的'tableView(_:cellForRowAtIndexPath:)'方法类似，当一些区域滑出屏幕外面的时候，map View也会返回一些重用的annotation View 的视图，所以这个代码第一次检查reusable annotation 是否存在，如果不存在，创建一个新的
3. 如果`MKAnnotationView `可以从缓存池子中获取到，则用Artwork的title和subtitle属性去设置它，并且决定显示callout，可以支持点击的功能-当点击这个pin大头针的时候会出现一个气泡效果



当然，你同样要设置

```
mapView.delegate = self
```
这样代理方法才会被调用到。

好了，运行程序：

![2](https://koenig-media.raywenderlich.com/uploads/2015/03/run32-174x320.png)

`mapView(_:viewForAnnotation:)`方法中，在弹出的气泡中，右边有一个详情按钮，可以设置点击它的处理事件，你可以弹出一个alert框，或者调到一个详情页面，
在这个教程中：当用户点击了这个按钮之后，我们弹出Maps的app,完成驾车/步行 等导航路线的功能，这很酷的!


## 加载Maps App
为了提高用户体验，打开`Artwork.swift`，并且添加如下代码，导入 AddressBook(地址薄)框架

```
import AddressBook
```

这里添加了一个`AddressBook`框架，你可能会问，这个地址簿框架是做什么的，和Map有什么关系呢？well,它包含了一些字典的键，比如`kABPersonAddressStreetKey`,当你设置一些地址或者城市或者州 的地址坐标时会用到它。


下一步，添加下面的帮助方法:

```
// annotation callout info button opens this mapItem in Maps app
func mapItem() -> MKMapItem {
  let addressDictionary = [String(kABPersonAddressStreetKey): subtitle]
  let placemark = MKPlacemark(coordinate: coordinate, addressDictionary: addressDictionary)
 
  let mapItem = MKMapItem(placemark: placemark)
  mapItem.name = title
 
  return mapItem
}
```

这里你从一个`MKPlacemark `中创建了一个`MKMapItem`，这个Maps app是可以从`MKMapItem`中获取到并且展示出来

下一步，你必须告诉MapKit当你点击 callout button 的时候，你要做什么

打开ViewController,添加`MKMapViewDelegate `的另外一个协议方法:`func mapView(mapView: MKMapView!, annotationView view: MKAnnotationView!, 
    calloutAccessoryControlTapped control: UIControl!)`
    
    
```
 func mapView(mapView: MKMapView!, annotationView view: MKAnnotationView!, 
    calloutAccessoryControlTapped control: UIControl!) {
  let location = view.annotation as! Artwork
  let launchOptions = [MKLaunchOptionsDirectionsModeKey: MKLaunchOptionsDirectionsModeDriving]
  location.mapItem().openInMapsWithLaunchOptions(launchOptions)
}
```

当用户点击地图上的大头针，弹出一个气泡视图，然后点击气泡视图上的info button,当用户点击这个info button的时候，`mapView(_:annotationView:calloutAccessoryControlTapped:) `该代理方法被调用

在这个方法中，你获取到Artwork对象，然后通过创建一个关联的`MKMapItem `，然后调用mapItem的`openInMapsWithLaunchOptions`方法去加载一个 `Maps app`


注意，你传递了一个字典给这个方法，这里允许你去指定几个不同的选项，这里的`DirectionModeKeys `设置成了`Driving`，这个就是显示的自驾模式，从你的当前定位地点到这个大头针的位置，当然，你可以设置其他的选项，比如walking,bus等模式。

我建议你通过传递在字典参数中传递其他几个不同的选项，看看展现的效果，你可以去查看`MKMapItem `类，这个类同事允许你传递多了可选项参数

在你运行程序之后，最后在xcode中设置一下你的坐标，打开`Product\Scheme\Edit Scheme`,然后选择`Run`,打开`Options`标签，选择`Core Location: Allow Location Simulation `,选择`Honolulu `

![1](https://koenig-media.raywenderlich.com/uploads/2014/12/defaultUserLocation-480x232.png)


好了，运行程序，你将会看到一个地图，点击地图上的大头针，然后点击info button,将会加载一个 Maps App

![1](https://koenig-media.raywenderlich.com/uploads/2015/03/run4b-174x320.png)

如果你也像我一样打开了地图，恭喜你！


## 解析JSon数据并且转化成Artwork对象

现在你知道如何在地图上展示artwork,怎嘛一股脑加载一个Maps app通过点击大头针的 callout info button,接下来我们介绍如何解析数据集。之后你可以从本地的地图区域中搜索数据。

首先，找到`JSON.swift`和文件：`PublicArt.json.`。`JSON.swift`包含了很好的解析json数据的API,提提供了对每个JSON value数据的转化。

把这两个文件拖入到工程中，在`Artwork.swift`添加如下方法:

```
class func fromJSON(json: [JSONValue]) -> Artwork? {
  // 1
  var title: String
  if let titleOrNil = json[16].string {
    title = titleOrNil
  } else {
    title = ""
  }
  let locationName = json[12].string
  let discipline = json[15].string
 
  // 2
  let latitude = (json[18].string! as NSString).doubleValue
  let longitude = (json[19].string! as NSString).doubleValue
  let coordinate = CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
 
  // 3
  return Artwork(title: title, locationName: locationName!, discipline: discipline!, coordinate: coordinate)
}
```
代码解析:

1. `fromJSON‘s `方法将会从一个数组中返回一个Artwork对象，如果你传递一个数组的元素，你将会看到title, locationName等字段，在这个方法中都有指定的索引来获取数据，这`title`对某些artwork来说又能是空的(nil).
2. 转换经纬度为double
3. 返回artwork对象

返回的json数据大概是这样:

```
[ 55, "8492E480-43E9-4683-927F-0E82F3E1A024", 55, 1340413921, "436621", 1340413921, "436621", "{\n}", "Sean Browne",
"Gift of the Oahu Kanyaku Imin Centennial Committee", "1989", "Large than life-size bronze figure of King David Kalakaua
mounted on a granite pedestal. Located at Waikiki Gateway Park.", "Waikiki Gateway Park", 
"http://hiculturearts.pastperfect-online.com/34250images/002/199103-3.JPG", "1991.03", "Sculpture", "King David 
Kalakaua", "Full", "21.283921", "-157.831661", [ null, "21.283921", "-157.831661", null, false ], null ]
```

在`ViewController.swift`中调用` fromJSON(_:)`方法，然后添加一个数组去存储从数据集中读到的json数据

```
var artworks = [Artwork]()
```
然后，添加下面的help方法:

```
func loadInitialData() {
  // 1
  let fileName = NSBundle.mainBundle().pathForResource("PublicArt", ofType: "json");
  var readError : NSError?
  var data: NSData = NSData(contentsOfFile: fileName!, options: NSDataReadingOptions(0),
    error: &readError)!
 
  // 2
  var error: NSError?
  let jsonObject: AnyObject! = NSJSONSerialization.JSONObjectWithData(data, 
    options: NSJSONReadingOptions(0), error: &error)
 
  // 3
  if let jsonObject = jsonObject as? [String: AnyObject] where error == nil,
  // 4
  let jsonData = JSONValue.fromObject(jsonObject)?["data"]?.array {
    for artworkJSON in jsonData {
      if let artworkJSON = artworkJSON.array,
      // 5
      artwork = Artwork.fromJSON(artworkJSON) {
        artworks.append(artwork)
      }
    }
  }
}
```

## 标注Artworks

现在你已经有了一组artwork的数据集，现在你需要把它们加入到地图上。

在`ViewDidLoad`方法中，创建`initialLocation `,然后设置map的中心点，调用`loadInitialData()`和`mapView.addAnnotations(artworks):`

```
loadInitialData()
mapView.addAnnotations(artworks)
```

删除之前创建的`King David Kalakaua`地图注解 数据的代码，你现在不需要它们了

```
//    let artwork = Artwork(title: "King David Kalakaua", locationName: "Waikiki Gateway Park",
//      discipline: "Sculpture", coordinate: CLLocationCoordinate2D(latitude: 21.283921, longitude: -157.831661))
//    mapView.addAnnotation(artwork)
```

编译运行程序，你的app可以看到很多的大头针

![2](https://koenig-media.raywenderlich.com/uploads/2015/03/run5a1-174x320.png)

## 为大头针设置不同的颜色

还记的在artwork中的`discipline `属性吗？
它的值像`Sculpture, Plaque, Mural, Monument, other`

在`Artwork.swift`中添加如下方法:

```
// pinColor for disciplines: Sculpture, Plaque, Mural, Monument, other
func pinColor() -> UIColor  {
        switch discipline {
        case "Sculpture", "Plaque":
            return UIColor.red
        case "Mural", "Monument":
            return UIColor.purple
        default:
            return UIColor.green
        }
    }
```

然后在ViewController中找到`mapView(_:viewForAnnotation:) `代码方法，设置pinColor属性:

```
view.pinTintColor = annotation.pinColor()

```

## 使用定位权限

想要获取定位服务，首先要导入 `Corelocation`框架

在ViewController中加入下面代码:

```
// MARK: - location manager to authorize user location for Maps app
var locationManager = CLLocationManager()
func checkLocationAuthorizationStatus() {
  if CLLocationManager.authorizationStatus() == .AuthorizedWhenInUse {
    mapView.showsUserLocation = true
  } else {
    locationManager.requestWhenInUseAuthorization()
  }
}
 
override func viewDidAppear(animated: Bool) {
  super.viewDidAppear(animated)
  checkLocationAuthorizationStatus()
}
```

## 在info.plist中需要加入权限设置
这里有几个权限设置你需要添加，你不加入的话，你的程序不会崩溃，但是你的locationManager’s的请求将会无效。为了让其工作良好，你必须添加`NSLocationWhenInUseUsageDescription `在你的app's的`Information Property Lis`,设置它的类型是string,值为一个消息，这个消息解释了为什么用户需要允许app访问他们的位置信息。

打开 `Info.plist`，点开左边的小三角
![1](https://koenig-media.raywenderlich.com/uploads/2015/01/Infoplist-480x190.png)

最好再添加两个选线 Privacy 开头的和定位信息有关：
三个权限设置如下:

1. Privacy - Location When In Use Usage Description
2. Privacy - Location Usage Description
3. Privacy - Location Always Usage Description



## Final project

[swift3版本的工程](https://koenig-media.raywenderlich.com/uploads/2015/03/HonoluluArt-Swift3.zip)下载
