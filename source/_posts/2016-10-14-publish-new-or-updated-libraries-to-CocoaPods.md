---
title: 手把手教你发布代码到CocoaPods
date: 2016-10-14 16:00:00
tags: 
     - CocoaPods
     - iOS
categories: iOS
keywords: CocoaPods
description: 详细介绍了如何发布代码到CocoaPods，以及后续的更新等
---

# 目录
- 前言
- CocoaPods原理
- 发布与更新自己的pod
  - 安装CocoaPods
  - 创建编辑自己的pod
  - 本地初始化pod并提交到Github
  - 注册CocoaPods并提交pod
  - 更新自己的pod
- 其他命令

# 前言

最近App在整理一些第三方库，包括一些没怎么再维护的替换为自己实现，所以做了一些简单的库上传到CocoaPods方便管理和维护。



# CocoaPods原理

每种语言发展到一定阶段，就会出现相应的依赖管理工具，例如Node.js的*npm*、Swift的*Swift Package Manager*，早期为iOS提供依赖管理的工具叫做：CocoaPods。

特点：

+ CocoaPods会将所有的第三方库以target的方式组成一个名为Pods的工程。整个第三方库工程会生成一个库提供给我们使用。
+ 我们的工程和Pods工程会被以workspace的形式组织和管理，也就是**.xcworkspace*文件。

好处：

+ CocoaPods上有很多好用的库，以此缩短我们的开发周期和提升软件的质量。
+ CocoaPods会自动将第三方开源库的源码下载下来，并且为工程设置好相应的系统依赖和编译参数。
+ 通过*Podfile*可以很方便地更新第三方库。




# 发布与更新自己的pod

下面以我提交的库*ZInAppPurchase*为例，讲解下整个过程和遇到的问题。

> 参考官方文档[Making CocoaPods](http://guides.cocoapods.org/making/index.html)



## 安装CocoaPods

CocoaPods是用Ruby实现的，而Gem是一个管理Ruby库和程序的标准包，它通过Ruby Gem源来查找、安装、升级和卸载软件包。

+ 更新Gem

如果没有翻墙可能会比较慢，可以将原来的源替换成新的。
```
// 查看本机当前设置的源
ZackZhengdeMacBook-Pro:~ zack$ gem source -l
*** CURRENT SOURCES ***

https://rubygems.org/
// 移除掉原有的源
ZackZhengdeMacBook-Pro:~ zack$ gem sources --remove https://rubygems.org/
// 添加国内的源
ZackZhengdeMacBook-Pro:~ zack$ gem sources -a https://ruby.taobao.org/
```
> **淘宝的源已不再维护**，可以使用新的源，参考[RubyGems 镜像 - Ruby China](https://gems.ruby-china.org/)

接下来更新Gem：
```
ZackZhengdeMacBook-Pro:~ zack$ sudo gem update --system
```

+ 安装并初始化CocoaPods

```
ZackZhengdeMacBook-Pro:~ zack$ sudo gem install cocoapods --pre
Password:
Successfully installed cocoapods-1.1.0.rc.3
Parsing documentation for cocoapods-1.1.0.rc.3
Done installing documentation for cocoapods after 2 seconds
1 gem installed
```

使用  --pre 是为了尽量使用最新的版本，没更新时遇到过`pod lib lint`发生下面错误的情况：

```
xcodebuild: error: 'App.xcworkspace' does not exist.
```

初始化：

```
ZackZhengdeMacBook-Pro:~ zack$ pod setup
Setting up CocoaPods master repo
  $ /usr/bin/git -C /Users/zack/.cocoapods/repos/master pull --ff-only
  From https://github.com/CocoaPods/Specs
     cae692c..3231da7  master     -> origin/master
  Updating cae692c..3231da7
  Fast-forward
  ......
Setup completed
```

> 所有项目的Podspec文件都托管在 https://github.com/CocoaPods/Specs 第一次执行 pod setup时，CocoaPods会将这些podspec索引文件更新到本地的 ~/.cocoapods/目录下，这个索引文件比较大。网上有讲解更换repo镜像的方法。




## 创建编辑自己的pod


使用`pod lib`的命令。

```
ZackZhengdeMacBook-Pro:~ zack$ pod lib create ZInAppPurchase
Cloning `https://github.com/CocoaPods/pod-template.git` into `ZInAppPurchase`.
Configuring ZInAppPurchase template.

------------------------------

To get you started we need to ask a few questions, this should only take a minute.

If this is your first time we recommend running through with the guide: 
 - http://guides.cocoapods.org/making/using-pod-lib-create.html
 ( hold cmd and double click links to open in a browser. )


What language do you want to use?? [ Swift / ObjC ]
 > Swift

Would you like to include a demo application with your library? [ Yes / No ]
 > Yes

Which testing frameworks will you use? [ Quick / None ]
 > None

Would you like to do view based testing? [ Yes / No ]
 > No

Running pod install on your new library.

Analyzing dependencies
Fetching podspec for `ZInAppPurchase` from `../`
Downloading dependencies
Installing ZInAppPurchase (0.1.0)
Generating Pods project
Integrating client project

[!] Please close any current Xcode sessions and use `ZInAppPurchase.xcworkspace` for this project from now on.
Sending stats
Pod installation complete! There is 1 dependency from the Podfile and 1 total pod installed.

 Ace! you're ready to go!
 We will start you off by opening your project in Xcode
  open 'ZInAppPurchase/Example/ZInAppPurchase.xcworkspace'

To learn more about the template see `https://github.com/CocoaPods/pod-template.git`.
To learn more about creating a new pod, see `http://guides.cocoapods.org/making/making-a-cocoapod`.
```

修改文件：

/Users/zack/ZInAppPurchase/ZInAppPurchase/Classes/ReplaceMe.swift
/Users/zack/ZInAppPurchase/ZInAppPurchase.podspec

>  **.podspec*文件主要是修改*s.homepage*、*s.source*、*s.frameworks*。

*ZInAppPurchase.podspec*内容如下：

```
Pod::Spec.new do |s|
  s.name             = 'ZInAppPurchase'
  s.version          = '0.1.1'
  s.summary          = 'A short description of ZInAppPurchase.'

  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://github.com/sapphirezzz/ZInAppPurchase'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'sapphirezzz' => 'zhengzuanzhe@gmail.com' }
  s.source           = { :git => 'https://github.com/sapphirezzz/ZInAppPurchase.git', :tag => s.version.to_s }

  s.ios.deployment_target = '8.0'

  s.source_files = 'ZInAppPurchase/Classes/**/*'
  s.frameworks = 'Foundation', 'StoreKit'
end
```

安装pod：

```
ZackZhengdeMacBook-Pro:~ zack$ cd ZInAppPurchase/Example/
ZackZhengdeMacBook-Pro:Example zack$ pod install
Analyzing dependencies
Fetching podspec for `ZInAppPurchase` from `../`
Downloading dependencies
Installing ZInAppPurchase 0.1.0 (was 0.1.0)
Generating Pods project
Integrating client project
Sending stats
Pod installation complete! There is 1 dependency from the Podfile and 1 total pod installed.
```

> 此时会看到工程目录下多出ZInAppPurchase.xcworkspace、Podfile.lock文件和Pods目录。



接下来可以打开下面文件编写库：

/Users/zack/ZInAppPurchase/Example/ZInAppPurchase.xcworkspace



## 本地初始化pod并提交到Github


先到Github建立仓库*ZInAppPurchase*，并提交代码：

```
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ git add -A && git commit -m "Release 0.1.0"
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ git tag '0.1.1'
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ git remote add origin git@github.com:sapphirezzz/ZInAppPurchase.git
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ git push --tags
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ git push origin master
```

初始化pod：

```
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ pod lib lint --allow-warnings

 -> ZInAppPurchase (0.1.0)
    - WARN  | summary: The summary is not meaningful.

ZInAppPurchase passed validation.
```

> 使用*--allow-warnings*是使得有warning时也能验证通过。比如：WARN  | summary: The summary is not meaningful.



## 注册CocoaPods并提交pod

先注册：

```
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ pod trunk register zhengzuanzhe@gmail.com 'zackzheng' --description='macbook pro'
[!] Please verify the session by clicking the link in the verification email that has been sent to zhengzuanzhe@gmail.com
```

然后打开收到的标题为*[CocoaPods] Confirm your session.*的邮件，点击链接之后跳转页面，会看到：

*You can go back to your terminal now.*

最后就deploy了：

```
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ pod trunk push ZInAppPurchase.podspec --allow-warnings
Updating spec repo `master`
Validating podspec
 -> ZInAppPurchase (0.1.0)
    - WARN  | summary: The summary is not meaningful.

Updating spec repo `master`
  - Data URL: https://raw.githubusercontent.com/CocoaPods/Specs/ae961ca47afb2a9171653382214897c4e160a679/Specs/ZInAppPurchase/0.1.0/ZInAppPurchase.podspec.json
  - Log messages:
    - October 13th, 11:28: Push for `ZInAppPurchase 0.1.0' initiated.
    - October 13th, 11:28: Push for `ZInAppPurchase 0.1.0' has been pushed
    (1.423909231 s).
```

看看库是否已经可以找到：

```
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ pod search ZInAppPurchase
-> ZInAppPurchase (0.1.0)
   A short description of ZInAppPurchase.
   pod 'ZInAppPurchase', '~> 0.1.0'
   - Homepage: https://github.com/sapphirezzz/ZInAppPurchase
   - Source:   https://github.com/sapphirezzz/ZInAppPurchase.git
   - Versions: 0.1.0 [master repo]
(END)
```

> 组合键shift+Q退出



## 更新自己的pod


修改完代码之后，提交到Github并push到CocoaPods：


```
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ git add -A && git commit -m 
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ git tag '0.1.1'
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ git push --tags
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ pod trunk push ZInAppPurchase.podspec --allow-warnings
Updating spec repo `master`
Validating podspec
 -> ZInAppPurchase (0.1.1)
    - WARN  | summary: The summary is not meaningful.

Updating spec repo `master`
  - Data URL: https://raw.githubusercontent.com/CocoaPods/Specs/1fba9c650a4212c0b7be707698b8597556f0ece7/Specs/ZInAppPurchase/0.1.1/ZInAppPurchase.podspec.json
  - Log messages:
    - October 13th, 12:42: Push for `ZInAppPurchase 0.1.1' initiated.
    - October 13th, 12:43: Push for `ZInAppPurchase 0.1.1' has been pushed
    (1.090057228 s).
```



# 其他命令

查看CocoaPods上的个人信息：

```
ZackZhengdeMacBook-Pro:ZInAppPurchase zack$ pod trunk me
  - Name:     Zackzheng
  - Email:    zhengzuanzhe@gmail.com
  - Since:    May 30th, 21:28
  - Pods:
    - ZInAppPurchase
    - ZPlaceholderTextView
    - ZPopTipView
  - Sessions:
    - May 30th, 21:28       -        October 6th, 00:10. IP: 14.18.48.167 
    
    - September 28th, 02:06 - February 18th, 2017 10:43. IP: 121.33.185.113
    Description: macbook pro
    - October 13th, 11:24   - February 18th, 2017 11:31. IP: 183.240.19.242
    Description: macbook pro
```

有时候如果提交有问题，可以执行一下命令删除：

// 直接废去这个pod

pod trunk deprecate ZInAppPurchase

// 废去这个pod的某个版本

pod trunk delete ZInAppPurchase 1.0.0

> 测试过，两个命令使用完都没返回成功，且*pod search*是可以找到的，但实际却是成功的。



-END-
欢迎到我的博客交流：https://zackzheng.info