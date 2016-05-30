---
layout: post
title: "为iOS建立Travis CI"
date: 2016-05-23 14:30:30 +0800
comments: true
categories: ios
---

你是否曾经试着为ios项目搭建一台支持[持续集成](http://baike.baidu.com/link?url=9m9zB2m909wrZ82wdNWMhtPBTt14PduTPIyi78YL0sS7ccrE-DN22S55wCuEhuobFZ0dL5T4MhKmQzFLx1q4-K)的服务器，从我的个人经验而言，这可不是一个轻松的活。首先需要准备一台MAc电脑，并安装好全部所需的软件和插件。你要负责管理所有的用户账户，并提供安全保护。你需要授予访问仓库的权限，并配置所有的编译步骤和证书，在项目的运行时期，你需要保持服务器稳健和最新。

<!--more-->

最后，原本你想节省时间，会发现你花费了大量的时间去维护这台服务器，不过如果你的项目托管在github上，现在有了新的希望：[Travis CI](https://travis-ci.org/).该服务可以为你的项目提供持续集成的支持，也就意味着它会负责好托管一个项目的所有细节。在ruby的世界中，Travis CI以久负盛名。从2013年4月起，Travs也开始支持ios和mac平台

在这篇文章中，我将向你展示如何一步步的在项目中集成Travis.不仅包括项目的编译的单元测试运行，还包括将应用部署到你所有的测试设备上。

## GITHub集成

我最喜欢Travis的一点就是它与GitHub的webUI集成的非常好，例如pull请求，Travis会为每次请求都执行编译操作。如果一切正常，pull请求在Github上看起来这样:
![e](http://7xkxhx.com1.z0.glb.clouddn.com/github_ready_to_merge.jpg)

万一编译不成功，Github页面会修改相应的颜色。给予提醒:
![1](http://7xkxhx.com1.z0.glb.clouddn.com/github_merge_with_caution.jpg)


## 链接 Travis 和 GitHub
让我们看一下如何将Github项目与Travis链接上，使用Github账号登录 [Travis 站点](https://travis-ci.org/)。对于私有仓库，需要注册一个Travis 专业版账号

登录成功后，需要为项目开启Travis支持，导航到[属性页面](https://travis-ci.org/profile),该页面列出了所有的github项目，不过要注意，如果此后创建了一个新的仓库，要使用 sync now 按钮进行同步。Travis只会偶尔更新的项目列表.

![test](http://7xkxhx.com1.z0.glb.clouddn.com/objc_travis_flick.jpg)

现在只需要打开这个开关就可以在你的项目添加Travis服务，之后你会看到Travis会和Github项目设置关联。下一步就是告诉Travis，当它收到项目改动通知之后该做什么。

## 最简单的项目配置

Tranvis CI 需要项目的一些基本信息。在项目的根目录创建一个名为 .travis.yml的文件，文件中内容如下：


```
language: objective-c
```

Travis 编译器运行在虚拟机环境下，该编译器已经利用 `Ruby`,`homebrew`,`CocoaPods`和一些默认的编译脚本进行过[预配置](http://about.travis-ci.org/docs/user/osx-ci-environment/).上述的配置项已经足够编译你的项目了。

预装的编译脚本会分析你的Xcode项目，并对每个target进行编译。如果所有文件都没有编译错误，并且测试也没有被打断，那么项目就编译成功了。现在可以将改动push到GitHub上看看能否成功编译。

虽然上述配置过程真的很简单，不过对你的项目不一定适用。这里几乎没有什么文档来指导用户如何配置默认的编译行为。例如：有一次我没有用 iphonesimulator SDK 导致代码签名错误。如果刚刚那个最简单的配置对你的项目不适用的话，让我们来看一下如何对Travis使用自定义的编译命令

### 自定义编译命令
Travis 使用命令行对项目进行编译。因此，第一步就是使项目能够在本地编译。作为Xcode命令行工具的一部分，Apple提供了`xcodebuild`命令

打开终端并输入:

```
xcodebuild --help
```
上述命令会列出xcodebuild所有可用的参数。如果命令执行失败了，确保命令行工具已经成功安装。一个常见的编译命令看起来是这样的：

```
'xcodebuild -project {project}.xcodeproj -target {target} -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO
```
使用 iphonesimulator SDK是为了避免签名错误，知道我们稍后引入证书之前，这一步是必须的。通过设置`ONLY_ACTIVE_ARCH=NO`我们可以确保利用模拟器架构编译工程。你也可以设置额外的属性，例如`configuration `,输入 man xcodebuild查看相关文档。

对于使用 CocoaPods的项目，需要用下面的命令来指定 workspace和scheme:

```
xcodebuild -workspace {workspace}.xcworkspace -scheme {scheme} -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO

```

schemes 是由Xcode自动生成的，但这在服务器上不会发生。确保所有的scheme都被设置为 shared并加入到仓库中，否则它只会在本地工作而不会被Travis CI识别。

![w](http://7xkxhx.com1.z0.glb.clouddn.com/objc_shared_schemes.jpg)

我们实例项目下的 .travis.yml文件现在看起来应该是这样


```
language: objective-c
script: xcodebuild -workspace TravisExample.xcworkspace -scheme TravisExample -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO
```


## 运行测试
对月测试来说，通常使用如下这个命令(注意 test 属性)

```
xcodebuild test -workspace {workspace}.xcworkspace -scheme {test_scheme} -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO

```

不幸的是，xcodebuild对于ios来说，并不能正确支持target和应用程序的测试。[这里有一些解决方案](http://www.raingrove.com/2012/03/28/running-ocunit-and-specta-tests-from-command-line.html)不过我建议使用Xctool.

### Xctool
[Xctool](https://github.com/facebook/xctool)是来自FaceBook的命令行工具，它可以简化程序的编译和测试。它的彩色输出信息比 xcodebuild更加简洁美观。同时还添加了对逻辑测试，应用测试的支持。

Travis中已经预装了xctool。要在本地测试的话，需要用homebrew安装xctool:

xctool用法非常简单，它使用的参数跟xcodebuild相同：

```
xctool test -workspace TravisExample.xcworkspace -scheme TravisExampleTests -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO

```

一旦相关命令在本地能正常工作，那么就是时候把它们添加到 .travis.yml中了：


```
language: objective-c
script:
  - xctool -workspace TravisExample.xcworkspace -scheme TravisExample -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO
  - xctool test -workspace TravisExample.xcworkspace -scheme TravisExampleTests -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO
```

到此为止，介绍的内容对于使用Travis的library工程来说，已经足够了。我们可以确保项目正常编译并测试通过。但对于ios应用来说，我们希望能在真实的物理设备上进行测试。也就说我们需要将应用部署到我们的所有测试设备上。当然，我们希望Travis能自动完成这项任务。首先，我们需要给程序签名

# 应用程序的签名

为了在Travis中能给程序签名，我们需要准备好所有必须的证书和配置文件。就像每个ios开发人员指导的那样，这可能是最困难的一步。后面，我将写一些脚本在服务器上给应用程序签名。

## 证书和配置文件

### 1.苹果全球开发者关系认证
从[苹果官网](http://developer.apple.com/certificationauthority/AppleWWDRCA.cer)或者从钥匙串中导出。并将保存到项目的目录` scripts/certs/apple.cer`中。

### 2.iPhone发布证书 + 私钥
如果还没有发布证书的话，先创建一个。登录[苹果开发者账号](https://developer.apple.com/account/overview.action),按照步骤，创建一个新的生产环境证书(*Certificates*)>*Production*>*Add*>*App Store and ad hoc*)。然后下载并安装证书。之后，可以在钥匙串中找到它。打开Mac中的钥匙串应用程序：

![key](http://7xkxhx.com1.z0.glb.clouddn.com/dist_cert_keychain.jpg)

右键单击证书，选择`Export ....`将证书导出至`scripts/certs/dist.cer`.然后导出私钥并保存至`cripts/certs/dist.p12`.记得输入私钥的密码.

由于Travis需要知道私钥密码，因此我们要把这个密码存储在某个地方，当然，我们不希望已明文的形式存储。我们可以用[Travis的安全环境变量](http://about.travis-ci.org/docs/user/build-configuration/#Secure-environment-variables)。打开终端，并定位到包含 .travis.yml文件所在目录。首先用 `gem install travis`命令安装Travis gem.之后用下面的命令添加秘钥密码:

```
travis encrypt "KEY_PASSWORD={password}" --add
```

上面的命令会安装一个叫做`Key_Password`的加密环境变量到 .travis.yml 配置文件中。这样就可以在被 Travis CI执行的脚本中使用这个变量

### 3.ios 配置文件(发布)
如果还没有用于发布的配置文件，那么也创建一个新的。根据开发者账号类型，可以选择创建 [ADHoc](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/TestingYouriOSApp/TestingYouriOSApp.html)或[IN House](https://developer.apple.com/programs/ios/enterprise/gettingstarted/)配置文件*(Provisioning Profiles > Distribution > Add > Ad Hoc or In House)*.然后将其下载保存到`scripts/profile/`目录。

由于Travis需要访问这个配置文件，所以我们需要将这个文件的名字存储为一个全局变量。并将其添加到.travis.yml文件的全局环境变量section中。例如，如果配置文件的名字是 `TravisExample_Ad_Hoc.mobileprovision`,那么按照如下进行添加:

```
env:
  global:
  - APP_NAME="TravisExample"
  - 'DEVELOPER_NAME="iPhone Distribution: {your_name} ({code})"'
  - PROFILE_NAME="TravisExample_Ad_Hoc"
```

上面还声明了两个环境变量。第三行中的`APP_name`通常为项目默认的target的名字。第四行的`DEVELOPER_NAME`是xcode中，默认target里面Build Settings的Code Signing Identity > Release 对应的名字。然后搜索程序的`Ad Hod`或者`In house`配置文件，将其中黑体文字取出，根据设置的不同，括弧中可能不会有任何信息。

## 加密证书和配置文件
如果你的GitHub仓库是公开的，你可能希望对证书和配置文件进行加密。如果你的是私有仓库，可以跳过这一节


首先，我们需要一个密码来对所有的文件进行加密。在我们的实例中，密码为"foo",记住在你的工程中设置的密码应该更加复杂。在命令行汇总，我们使用`openssl`加密所有的敏感文件：

```
openssl aes-256-cbc -k "foo" -in scripts/profile/TravisExample_Ad_Hoc.mobileprovision -out scripts/profile/TravisExample_Ad_Hoc.mobileprovision.enc -a
openssl aes-256-cbc -k "foo" -in scripts/certs/dist.cer -out scripts/certs/dist.cer.enc -a
openssl aes-256-cbc -k "foo" -in scripts/certs/dist.p12 -out scripts/certs/dist.cer.p12 -a
```

通过上面的命令，可以创建出以 .enc结尾的加密文件。之后可以把原始文件忽略或者移除掉。至少不要把原始文件提交到GitHub中，否则原始文件会显示在GitHub中。如果你不小心把原始文件提交上去了，那么请看这里[如何解决](https://help.github.com/articles/remove-sensitive-data)

现在，我们的文件已经被加密了，接下来我们需要告诉Travis对文件进行解密。解密过程，需要用到密码。具体释放方法跟之前创建的`KEY_PASSOWORD`变量一样:

```
travis encrypt "ENCRYPTION_SECRET=foo" --add
```

最后，我们需要告诉Travis那些文件需要进行解密.将下面的命令添加到 .travis.yml文件中的before-script部分：


```
before_script:
- openssl aes-256-cbc -k "$ENCRYPTION_SECRET" -in scripts/profile/TravisExample_Ad_Hoc.mobileprovision.enc -d -a -out scripts/profile/TravisExample_Ad_Hoc.mobileprovision
- openssl aes-256-cbc -k "$ENCRYPTION_SECRET" -in scripts/certs/dist.p12.enc -d -a -out scripts/certs/dist.p12
- openssl aes-256-cbc -k "$ENCRYPTION_SECRET" -in scripts/certs/dist.p12.enc -d -a -out scripts/certs/dist.p12
```

就这样，在GitHub上面的文件就安全了，并且Travis依旧能读取并使用这些加密后的文件。但是有一个安全问题你需要知道，在Travis的编译日志中可能会显示出解密环境变量。不过对pull请求来说不会出现

## 添加脚本
现在我们需要确保证书都导入到了Travis CI的钥匙串中。为此，我们需要在scripts文件夹中添加一个名为 `add-key.sh`的文件：

```
#!/bin/sh
security create-keychain -p travis ios-build.keychain
security import ./scripts/certs/apple.cer -k ~/Library/Keychains/ios-build.keychain -T /usr/bin/codesign
security import ./scripts/certs/dist.cer -k ~/Library/Keychains/ios-build.keychain -T /usr/bin/codesign
security import ./scripts/certs/dist.p12 -k ~/Library/Keychains/ios-build.keychain -P $KEY_PASSWORD -T /usr/bin/codesign
mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
cp ./scripts/profile/$PROFILE_NAME.mobileprovision ~/Library/MobileDevice/Provisioning\ Profiles/
```

通过上面的命令创建了一个名为`ios-build`的临时钥匙串，里面包含了所有证书。注意，这里我们使用了`$Key_PASSWORD`来导入私钥。最后一步是将配置文件拷贝到LIbrary文件夹。

创建好文件之后，确保其授予了可执行的权限：在命令行输入:`chmod a+x scripts/add-key.sh `即可。为了正常使用脚本，必须要这样处理一下。

至此，已经导入了所有的证书和配置文件，我们可以开始给应用程序签名了。注意，在给程序签名之前必须对程序进行编译。由于我们需要知道编译结果存储在磁盘的具体位置，我建议在编译命令中使用`OBJROOT`和`SYMROOT`来指定输出目录，另外，为了创建 release版本，还需要把SDK设置为 iphones,以及configuration修改为 Release:

```
xctool -workspace TravisExample.xcworkspace -scheme TravisExample -sdk iphoneos -configuration Release OBJROOT=$PWD/build SYMROOT=$PWD/build ONLY_ACTIVE_ARCH=NO 'CODE_SIGN_RESOURCE_RULES_PATH=$(SDKROOT)/ResourceRules.plist'

```

如果运行了上面的命令，那么编译完成之后，可以在`build/Release-iphoneos`目录中找到对应程序的二进制文件。接下来，就可以对其签名，并创建IPA文件了。为此，我们创建了一个新的脚本：

```
#!/bin/sh
if [[ "$TRAVIS_PULL_REQUEST" != "false" ]]; then
  echo "This is a pull request. No deployment will be done."
  exit 0
fi
if [[ "$TRAVIS_BRANCH" != "master" ]]; then
  echo "Testing on a branch other than master. No deployment will be done."
  exit 0
fi

PROVISIONING_PROFILE="$HOME/Library/MobileDevice/Provisioning Profiles/$PROFILE_NAME.mobileprovision"
OUTPUTDIR="$PWD/build/Release-iphoneos"

xcrun -log -sdk iphoneos PackageApplication "$OUTPUTDIR/$APPNAME.app" -o "$OUTPUTDIR/$APPNAME.ipa" -sign "$DEVELOPER_NAME" -embed "$PROVISIONING_PROFILE"
```

第二行至第九行非常重要。我们并不希望在某个特性分支上创建新的release.对Pull请求也一样的。由于安全环境变量被禁用，所有pull请求也不会编译。


第十四行，才是真正的签名操作。这个命令会在`build/Release-iphoneos `目录下生成2个文件：`TravisExample.ipa`和`TravisExample.app.dsym`。第一个文件包含了分发至手机上的应用程序。dsym文件包含了二进制文件的调试信息。这个文件对于记录设备上的crash信息非常重要。之后当我们部署应用程序的时候，会用到这两个文件。


最后一个脚本是移除之前创建的临时钥匙串，并删除配置文件。虽然这不是必须的，不过这有助于进行本地测试。


```
#!/bin/sh
security delete-keychain ios-build.keychain
rm -f ~/Library/MobileDevice/Provisioning\ Profiles/$PROFILE_NAME.mobileprovision
```

最后一步，我们必须告诉Travis什么时候执行这三个脚本，在应用程序编译，签名和清楚等之前，需要先添加私钥。在 .travis.yml文件中添加如下内容：


```
before_script:
- ./scripts/add-key.sh
- ./scripts/update-bundle.sh
script:
- xctool -workspace TravisExample.xcworkspace -scheme TravisExample -sdk iphoneos -configuration Release OBJROOT=$PWD/build SYMROOT=$PWD/build ONLY_ACTIVE_ARCH=NO
after_success:
- ./scripts/sign-and-upload.sh
after_script:
- ./scripts/remove-key.sh
```

完成上面的所有操作之后，我们就可将所有内容push到GItHub上，等待Travis对应用程序进行签名，我们可以在工程页面下的Travis控制台验证是否一切正常，如果一切正常的话，下面来看看如何将签名好的应用程序部署给测试人员。

## 部署应用程序

这里有两个知名的服务可以帮你发布应用程序:[TestFlight](http://testflightapp.com/)和[HockyeyApp](http://hockeyapp.net/)。不管选择哪个都能满足需求。就我个人而言，推荐使用 HockeyApp,不过这里我会对这两个服务都做介绍。

首先我们队 sign-and-build.sh脚本做一个扩充--在里面添加一些release记录:

```
RELEASE_DATE=`date '+%Y-%m-%d %H:%M:%S'`
RELEASE_NOTES="Build: $TRAVIS_BUILD_NUMBER\nUploaded: $RELEASE_DATE"
```

注意这里使用了一个Travis的全局变量`TRAVIS_BUILD_NUMBER `.

## TestFlight
创建一个[TestFlight账号](https://testflightapp.com/register/)，并配置好应用程序。为了使用TestFlight的API,首先需要获得[apitoken](https://testflightapp.com/account/#api)和[teamtoken](https://testflightapp.com/dashboard/team/edit/?next=/api/doc/)。再强调一次，我们需要确保它们是加密的。在命令行中执行如下命令:

```
travis encrypt "TESTFLIGHT_API_TOKEN={api_token}" --add
travis encrypt "TESTFLIGHT_TEAM_TOKEN={team_token}" --add
```

现在我们可以调用相应的API了。并将下面的内容添加到`sign-and-build.sh:`

```
curl http://testflightapp.com/api/builds.json \
  -F file="@$OUTPUTDIR/$APPNAME.ipa" \
  -F dsym="@$OUTPUTDIR/$APPNAME.app.dSYM.zip" \
  -F api_token="$TESTFLIGHT_API_TOKEN" \
  -F team_token="$TESTFLIGHT_TEAM_TOKEN" \
  -F distribution_lists='Internal' \
  -F notes="$RELEASE_NOTES"
```

千万不要使用verbose标记(-v)--这会暴露加密tokens.

## HOckeyApp
注册一个[HockeyApp账号](http://hockeyapp.net/plans)，并创建一个新的应用程序。然后在概述页面获取一个AppId.接下来，我们必须创建一个Api Token.打开[这个页面](https://rink.hockeyapp.net/manage/auth_tokens)并创建一个。如果你希望自动的将新版本部署给所有的测试人员，那么请选择 Full Access版本。

对App Id 和token进行加密:

```
travis encrypt "HOCKEY_APP_ID={app_id}" --add
travis encrypt "HOCKEY_APP_TOKEN={api_token}" --add
```
然后在sign-and-build.sh文件中调用相关的API:

```
curl https://rink.hockeyapp.net/api/2/apps/$HOCKEY_APP_ID/app_versions \
  -F status="2" \
  -F notify="0" \
  -F notes="$RELEASE_NOTES" \
  -F notes_type="0" \
  -F ipa="@$OUTPUTDIR/$APPNAME.ipa" \
  -F dsym="@$OUTPUTDIR/$APPNAME.app.dSYM.zip" \
  -H "X-HockeyAppToken: $HOCKEY_APP_TOKEN"
```

注意我们还上传了dsym文件。如果集成了TestFlight或HockeyAppSdk，我们可以立即收集到易读的crash报告。

## Travis故障排除
知道如何不通过直接访问编译环境就能找出问题是非常重要的。

在写本文的时候，还没有可以下载的虚拟机映像。如果Travis不能正常编译，首先试着在本地重现问题。在本地执行跟Travis相同的编译命令：


```
xctool ...
```

为了调试shell脚本，首先需啊哟定义环境变量。我的做法是创建一个新的shell脚本来设置所有的环境变量。记得将这个脚本添加到 .gitignore文件中--因为我们不希望将该文件公开暴露出去。支队示例工程来说， config.sh脚本文件看起来是这样的：


```
#!/bin/bash

# Standard app config
export APP_NAME=TravisExample
export DEVELOPER_NAME=iPhone Distribution: Mattes Groeger
export PROFILE_NAME=TravisExample_Ad_Hoc
export INFO_PLIST=TravisExample/TravisExample-Info.plist
export BUNDLE_DISPLAY_NAME=Travis Example CI

# Edit this for local testing only, DON'T COMMIT it:
export ENCRYPTION_SECRET=...
export KEY_PASSWORD=...
export TESTFLIGHT_API_TOKEN=...
export TESTFLIGHT_TEAM_TOKEN=...
export HOCKEY_APP_ID=...
export HOCKEY_APP_TOKEN=...

# This just emulates Travis vars locally
export TRAVIS_PULL_REQUEST=false
export TRAVIS_BRANCH=master
export TRAVIS_BUILD_NUMBER=0
```


为了暴露所有的环境变量，执行如下命令(确保config.sh是可执行的)

```
. ./config.sh
```

然后试着运行 `echo $APP_NAME`,以此检查脚本是否正确。如果正确的话，那么现在我们不用做任何修改，就能在本地运行所有的shell脚本了。


如果在本地得到的是不同的编译信息，那么可能是使用了不同的库和gems.尽量试着将配置信息设置和Travis VM 相同的信息。Travis在这里列出了其所有的安装的软件版本，你也可以在travis的配置文件中添加调试信息得到所有库文件的版本。

```
gem cocoapod --version
brew --version
xctool -version
xcodebuild -version -sdk
```

在本地安装好与服务器完全相同的软件之后，再重新编译项目。

如果获取的编译信息仍然不一样，试着将项目check out到一个新目录。并确保所有的缓存都已清空。每次编译程序时，Travis都会创建一个全新的虚拟机，所以不存在缓存的问题，但在你的本地机器上可能会出现。

一旦在本地重现出和服务器相同的错误，就可以开始调差具体问题了。当然导致问题的原因取决于具体问题。一般来说，通过Google都能找到引发问题的根源。

如果一个问题影响到了Travis上的其它项目，那么可能是Travis环境配置的原因。

## 点评
TravisCI跟市面上同类产品相比还是有一些限制。因为Travis运行在一个预先配置好的虚拟机上，因此必须为每次编译都安装一遍所有的依赖。这会花费一些额外的时间。不过Travis团队已经在着手提供一种缓存机制解决这个问题了。

在一定程度上，你会依赖于Travis所提供的配置。比如你只能使用Travis内置的Xcode版本进行编译。如果你本地使用的Xcode版本比较新，你的项目在服务器上可能无法编译通过。如果Travis能够为不同的Xcode版本都分别设置一个对应虚拟机会就好了。

对于复杂的项目来说，你可能希望把整个编译任务分为编译应用，运行集成测试等。这样你可以快速获得编译信息而不用等所有的测试都完后才能。目前Travis还没有直接支持有依赖的编译。

当项目被push到Github上时，travis会自动触发。不过编译动作不会立即触发，你的项目会被放到一个根据项目所用语言不同而不同的一个全局编译队列，不过专业版允许并发编译。

## 总结
Travis Ci提供了一个动能完整的持续集成环境，已进行应用程序的编译，测试和部署。对于开源项目来说，这项服务是完全免费的。很多社区都得益于GitHub强大的持续集成能力。

如果你还没有用过Travis.赶紧去试试吧，它棒极了！