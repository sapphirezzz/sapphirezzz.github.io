---
title: 为iOS项目适配iOS 9和Xcode 7
date: 2015-09-29 16:00:00
tags: 
     - iOS
categories: iOS
keywords: iOS
description: 为iOS项目适配iOS 9和Xcode 7
---

# 目录
- Bitcode选项
- Https处理
- 应用隐私控制
- 还有

# Bitcode选项

## 现象

编译时出现以下提示：

```
You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE), obtain an updated library from the vendor, or disable bitcode for this target. for architecture arm64
```

就是提示某个库必须使用Bitcode重新构建，或者关闭Bitcode的选项。

## 解决过程

最开始是提示友盟统计分析的库不支持，看了友盟网站说最新版支持了，而我是用CocoaPods安装的，所以更新下：

```
pod update UMengAnalytics-NO-IDFA
```

发现更新完还是v3.5.x的版本，试了几个方法还是没有作用，所以只有将它从podfile里面删除，然后下载最新的v3.6.5无IDFA版本，之后安装，没问题了。之后是提示了另一个库——美洽不支持Bitcode，在美洽的Q群中问了工程师，他们说下个版本才支持。

看来要等所有用到的第三方库都支持Bitcode才行，转了一圈，最终还是要把Bitcode选项关闭。方法：**进入项目的TARGETS下，在“Build Settings”下有一个“Enable Bitcode”选项，改为“NO”即可**

## 关于Bitcode

Bitcode是什么？看下官方文档[App Distribution Guide – App Thinning (iOS, watchOS)](https://developer.apple.com/library/prerelease/watchos/documentation/IDEs/Conceptual/AppDistributionGuide/AppThinning/AppThinning.html#//apple_ref/doc/uid/TP40012582-CH35-SW2)，里面可以找到定义:

(以下斜体字部分引用自[理解Bitcode：一种中间代码](http://www.cocoachina.com/ios/20150817/13078.html))：

*bitcode是被编译程序的一种中间形式的代码。包含bitcode配置的程序将会在App store上被编译和链接。bitcode允许苹果在后期重新优化我们程序的二进制文件，而不需要我们重新提交一个新的版本到App store上。*

*当我们提交程序到App store上时，Xcode会将程序编译为一个中间表现形式(bitcode)。然后App store会再将这个botcode编译为可执行的64位或32位程序。*

*再看看这两段描述都是放在App Thinning(App瘦身)一节中，可以看出其与包的优化有关。*

*如上面所说，bitcode是一种中间代码。LLVM官方文档有介绍这种文件的格式，有兴趣的可以移步LLVM Bitcode File Format。*

对于iOS，bitcode是可选的。对于watchOS，bitcode是必须的。Mac OS不支持bitcode。Xcode7默认会开启Bitcode。

# Https处理

## 现象

改完Bitcode，可以顺利编译通过了，运行，发现所有网络链接都没有返回数据。

```
The resource could not be loaded because the App Transport Security
```

## 解决过程

这个在iOS 9出来之前网上就一直在说，iOS9把所有的http请求都改为https了，说程序猿又要加班了什么的哈哈。

*如何适配？—弱弱地问下：加班要多久？*

两个解决方案：

1. 立即让公司的服务端升级使用TLS 1.2
2. 虽Apple不建议，但可通过在 Info.plist 中声明，倒退回不安全的网络请求依然能让App访问指定http，甚至任意的http

因为后台还没支持，所以临时解决先继续支持http请求，方法：**在 Info.plist 中添加 NSAppTransportSecurity ，类型是 Dictionary ，Dictionary 下添加 NSAllowsArbitraryLoads ，类型 Boolean ，值设为 YES。**

在[iOS 9 适配系列教程](http://www.cocoachina.com/ios/20150703/12392.html)一文中还讲了更严谨的做法，刚看到，明天试下。

## 关于Https

*iOS9系统发送的网络请求将统一使用TLS 1.2 SSL。采用TLS 1.2 协议，目的是 强制增强数据访问安全，而且系统 Foundation 框架下的相关网络请求，将不再默认使用 Http 等不安全的网络协议，而默认采用 TLS 1.2。*

关于Https的就不解释了，网上可以找到很多说明。

# 应用隐私控制

## 现象

刚好改了支付的网络层，测试下微信支付发现没法打开微信，然后看到在运行时输出栏就提示了以下信息：

```
2015-09-29 12:03:56.426 XXX[1199:510529] -canOpenURL: failed for URL: "wechat://" - error: "This app is not allowed to query for scheme wechat"
2015-09-29 11:52:51.933 XXX[1187:507850] -canOpenURL: failed for URL: "weixin://app/wx8dd9768db7127fdb/" - error: "This app is not allowed to query for scheme weixin"
2015-09-29 11:52:51.937 XXX[1187:507850] -canOpenURL: failed for URL: "mqqopensdkapiV2://qqapp" - error: "This app is not allowed to query for scheme mqqopensdkapiV2"
2015-09-29 11:52:51.938 XXX[1187:507850] -canOpenURL: failed for URL: "mqqOpensdkSSoLogin://qqapp" - error: "This app is not allowed to query for scheme mqqOpensdkSSoLogin"
2015-09-29 11:52:51.939 XXX[1187:507850] -canOpenURL: failed for URL: "mqq://qqapp" - error: "This app is not allowed to query for scheme mqq"
2015-09-29 11:52:51.940 XXX[1187:507850] -canOpenURL: failed for URL: "mqzoneopensdkapiV2://qzapp" - error: "This app is not allowed to query for scheme mqzoneopensdkapiV2"
2015-09-29 11:52:51.941 XXX[1187:507850] -canOpenURL: failed for URL: "mqzoneopensdkapi19://qzapp" - error: "This app is not allowed to query for scheme mqzoneopensdkapi19"
2015-09-29 11:52:51.942 XXX[1187:507850] -canOpenURL: failed for URL: "mqzoneopensdkapi://qzapp" - error: "This app is not allowed to query for scheme mqzoneopensdkapi"
2015-09-29 11:52:51.942 XXX[1187:507850] -canOpenURL: failed for URL: "mqzoneopensdk://qzapp" - error: "This app is not allowed to query for scheme mqzoneopensdk"
2015-09-29 11:52:51.943 XXX[1187:507850] -canOpenURL: failed for URL: "mqzone://qzapp" - error: "This app is not allowed to query for scheme mqzone"
```

## 解决过程

参照网上的，在”Info.plist”里面增加 LSApplicationQueriesSchemes 的键，类型是Array，然后把需要支持的 URL scheme 分别加在里面，类型String。主要是一些分享、收藏、支付、登录等等的应用。

我直接把上面输出栏提示的都加进去，如wechat、weixin、mqq等等。

## 关于隐私控制

[Quick Take on iOS 9 URL Scheme Changes](http://awkwardhare.com/post/121196006730/quick-take-on-ios-9-url-scheme-changes)

原来，在iOS9中，如果使用URL scheme必须在”Info.plist”中将你要在外部调用的URL scheme列为白名单，否则不能使用。

# 还有

## 现象

解决完可以编译通过，运行没问题，但是有两个warning：

```
All interface orientations must be supported unless the app requires full screen.
A launch storyboard or xib must be provided unless the app requires full screen.
```

## 解决过程

[iOS项目更新之升级Xcode7 & iOS9](http://www.tuicool.com/articles/FFbqmyr) 这篇文章是这么解释的：

*看到这句提示，就是说App默认是有开启了多任务功能，而多任务功能是需要App支持所有方向，如果我们App是有需要支持多任务，则需要开启App对各个方向（上、下、左、右）的支持;如果App不需要开启多任务，则只需要将如下示意图的 requires full screen 勾选上就ok（如图2.4）。*

requires full screen 勾选完运行，没有warning，并且对应用没什么影响。

## 关于分屏多任务

[iOS 9之分屏多任务（multitasking）](http://www.bubuko.com/infodetail-1022142.html) 这篇文章讲了一些，可以看看。



-END-
欢迎到我的博客交流：https://zackzheng.info