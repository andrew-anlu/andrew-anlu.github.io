---
layout: post
title: "Mac上如何把视频转成.Gif"
date: 2016-04-29 12:56:45 +0800
comments: true
categories: Mac
---
之前看到很多github上的开源项目都带.gif的演示效果，感觉很好，今天就试着制作了一下，今天把我的制作过程记录下来，方便他人参考。

<!--more-->

##录制视频
mac上自带的Quicktime就可以录制视频，选择录制屏幕

![1](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20160429-0.png)


录制完之后是 .mov 的后缀视频文件

##通过命令行转换

在苹果系统上，可以通过命令行的方式转换视频，首先假定你的机器上已经安装了[homebrew](http://www.brew.sh).

然后打开终端，安装[ffmpeg](http://baike.baidu.com/link?url=H-RabHoLq9tR0Zxn-jduRSC7NAlSBkEjjsRnXEJ7Rw9RFE6dUNBnYyXidoiXNGCyOWKdcAtQMg-J8x_hFPSg5K)插件

```
$ brew install ffmpeg
```

然后定位到正确的目录，（有视频文件的那个目录，比如 Desktop）,假定你的视频名字是`ScreenFlow.mov`,你运行下面的命令就能生成GIF:


```
$ ffmpeg -i ScreenFlow.mov -pix_fmt rgb24 output.gif
```
你会注意到生成文件很大，可以通过[ image magick](http://baike.baidu.com/link?url=WENMRejdDuoS2eXPiXrnvF2ohwpbTAe5oW_SOTsr2k099YbjyU4wCi-ngV31jzxvP0TvC-INM0FBcUk1sEDWQK)工具来减少大小

```
$ brew install imagemagick
```

运行:

```
$ convert -layers Optimize output.gif output_optimized.gif
```


##通过软件生成.GIF
[GifBrewery](http://gifbrewery.com)是一款GIF动态图片制作工具. 支持将视频文件剪裁输出成GIF动态图片。

并且支持裁剪视频的长度，本人感觉挺好用的。




##上传到云端

最后把制作好的.gif图片上传到云端就行了，比如 dropbox,七牛,阿里云等。
