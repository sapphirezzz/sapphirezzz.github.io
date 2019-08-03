---
title: iOS下实现Universal Links及相关问题小总结
date: 2016-01-26 16:00:00
tags: 
     - iOS
     - Universal Links
categories: iOS
keywords: iOS UniversalLinks
description: iOS下实现Universal Links及相关问题小总结
---

# 目录

- 前言
- Universal Links有什么用
- Universal Links特点
- Universal Links缺点
- 支持Universal Links
    - 服务器端
    - App端
- 流程

# 前言

最近研究了Universal Links，总结一下。

网上资料很少，基本都是苹果官方文档翻译过来的，所以直接看[官方文档](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html)就行了。

# Universal Links有什么用

> When you support universal links, iOS 9 users can tap a link to your website and get seamlessly redirected to your installed app without going through Safari. If your app isn’t installed, tapping a link to your website opens your website in Safari.

支持之后，只要点击一个指向你网站的链接，就会直接跳到你的App的页面，无需通过Safari。如果设备上没有安装你的应用，则会在Safari中打开你的网址。

> Universal links let iOS 9 users open your app when they tap links to your website within `WKWebView` and `UIWebView views` and `Safari pages`, in addition to links that result in a call to openURL:, such as those that occur in Mail, Messages, and other apps.

这里指明除了其他调用`openURL`的App外，**只有WKWebView、UIWebView、Safari内点击的才支持跳转**。像邮件、信息也是可以的，这个测试过。

> For users who are running versions of iOS earlier than 9.0, tapping a universal link to your website opens the link in Safari.

这功能**只支持iOS 9.0以上系统**，更早的系统版本会直接在Safari打开链接。

# Universal Links特点

- **Unique.** Unlike custom URL schemes, universal links can’t be claimed by other apps, because they use standard HTTP or HTTPS links to your website.
- **Secure.** When users install your app, iOS checks a file that you’ve uploaded to your web server to make sure that your website allows your app to open URLs on its behalf. Only you can create and upload this file, so the association of your website with your app is secure.
- **Flexible.** Universal links work even when your app is not installed. When your app isn’t installed, tapping a link to your website opens the content in Safari, as users expect.
- **Simple.** One URL works for both your website and your app.
- **Private.** Other apps can communicate with your app without needing to know whether your app is installed.

**总结下来是**：

1. 一个链接即可同时适用App和网站，在没安装时能继续打开网址不影响体验和用户预期。其他应用想使用时只需打开链接无需知道其他信息。
2. 相比URL Schemes，因为链接和路由是自己服务器指定，所以App和网站之间的联系相对安全。而URL Schemes的schemes是可获取可冒充的（*我抓包得知小红书的schemes为xhsdiscover，简单支持下就能使得原来跳转到小红书的链接跳转到我的App了*）。另外链接也具有唯一性，而URL Schemes不同应用申明时可相同。

# Universal Links缺点

- **在微信内置浏览器内无法跳转**。微信屏蔽了任何跳转到第三方应用的方式，包括URL Schemes（已测试）。但是使用发现小红书、58同城几个App可以，网上有提到微信WeixinJSBridge API（在小红书跳转抓包时发现的确用了`launch3rdApp`这个API，同事问了工作人员，说得去联系商务部，大概是合作后才能调用这个API）；另外内容分享到微信时可以选择分享类型为应用类型，这也可以打开应用，但是不能路由到App的指定页面。

- 只适用iOS 9以上系统版本。网上有个叫HOKO的好像可以支持iOS 5以上。

- **无法退回之前的页面**。我们总希望把用户留在App里面的，但是也许有些情况，之前的页面信息对用户很重要，这时只能切换到浏览器返回，无法在App里继续之前的页面。除非做多些额外工作。

  

# 支持Universal Links

需要服务器和客户端做些支持。

## 服务器端

- 配置apple-app-site-association文件

格式如下：

```
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "9JA89QQLNQ.com.apple.wwdc",
                "paths": [ "/wwdc/news/", "/videos/wwdc/2015/*", "NOT /videos/wwdc/2010/*", "/videos/wwdc/201?/*"]
            },
            {
                "appID": "TeamID.BundleID2",
                "paths": [ "*" ]
            }
        ]
    }
}
```

文件指明支持Universal Links的App和App所支持的路径。`appID`格式是`TeamID.BundleID`的；`paths`是个数组，用*表示支持所有路径，用`NOT`表示不支持的路径，还可以?适配。

**不支持的路径将会直接打开而不会调用App。**

- 将文件放到网站的根目录下。

> After you create the `apple-app-site-association` file, upload it to the root of your HTTPS web server. The file needs to be accessible via HTTPS—without any redirects—at `https:///apple-app-site-association`. 

需要注意的是，该网站必须**支持https访问**，并且证书有效。可以用这个链接[App Search API Validation Tool](https://search.developer.apple.com/appsearch-validation-tool/)测试。只要Universal Links那行状态为`PASSED`即可。

## App端

- 在 Xcode 的 `capabilities` -> `applinks`里添加支持的几个域名

都加上前置`applinks:`如`applinks:www.domain.com`。并且下面有红色感叹号时点击Fix。系统会自动写入.entitlements文件并添加到工程中。

- 处理接收到的Activity

在`UIApplicationDelegate`的`- application:continueUserActivity:restorationHandler:`里面处理，判断接收到的`NSUserActivity`的`activityType`值是否为`NSUserActivityTypeBrowsingWeb`，是的话则处理它的`webpageURL`，可以使用`NSURLComponents`处理。另外如果处理不了Apple建议最好openURL这个链接。

```
func application(application: UIApplication, continueUserActivity userActivity: NSUserActivity, restorationHandler: ([AnyObject]?) -> Void) -> Bool {
    if userActivity.activityType == NSUserActivityTypeBrowsingWeb {
        let webpageURL = userActivity.webpageURL! // Always exists
        if !isURLUnSupported(URL: webpageURL) {
            UIApplication.sharedApplication().openURL(webpageURL)
        }
    }
    return true
}
```

另外还有下面的说明。

> NOTE
>
> If you instantiate a [SFSafariViewController](https://developer.apple.com/library/prerelease/ios/documentation/SafariServices/Reference/SFSafariViewController_Ref/index.html#//apple_ref/doc/uid/TP40016220-CH1), [WKWebView](https://developer.apple.com/library/prerelease/ios/documentation/WebKit/Reference/WKWebView_Ref/index.html#//apple_ref/doc/uid/TP40014624-CH1), or [UIWebView](https://developer.apple.com/library/prerelease/ios/documentation/UIKit/Reference/UIWebView_Class/index.html#//apple_ref/doc/uid/TP40006950-CH3) object to handle a universal link, iOS opens your website in Safari instead of opening your app. However, if the user taps a universal link from within an embedded `SFSafariViewController`, `WKWebView`, or `UIWebView` object, iOS opens your app.

- 测试

可以直接调试测试，只要TeamID和BundleID一致。如在短信里或Safari里面点击支持的链接。

# 流程

1. 应用安装到设备后自动（有网友测试过）去拉取支持的每个域名下的`apple-app-site-association`文件。
2. 点击链接后，系统会检测该链接的域名和路径是否符合App的配置，符合则会打开App，调用接口`application:continueUserActivity:restorationHandler`方法；不符合或未安装该App则继续打开链接。
3. 判断`application:continueUserActivity:restorationHandler`接口获取到传入的`NSUserActivity`的`activityType`值是否为`NSUserActivityTypeBrowsingWeb`，是则处理`webpageURL`；
4. 如果`webpageURL`不符合App内支持的规则，建议openURL这个链接。



-END-
欢迎到我的博客交流：https://zackzheng.info