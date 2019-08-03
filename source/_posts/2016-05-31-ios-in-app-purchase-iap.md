---
title: iOS In-App Purchase(IAP) 流程与实现
date: 2016-05-31 16:00:00
tags: 
     - iOS
     - IAP
categories: iOS
keywords: iOS 内购 IAP
description: 主要介绍了 iOS In-App Purchase(IAP) 的流程和实现
---

## 目录

- 前言
- 程序内购买流程
  - iTunes Connect商品配置
  - 添加沙箱技术测试员
  - App内获取购买商品
  - 验证receipt
- 测试截图
- 提交审核
- 审核问题

## 前言

最近做了iOS程序内购买，封装了一下给上层调用，现在介绍下流程和简单的实现。具体可以看我上传到Github的代码[ZInAppPurchase](https://github.com/sapphirezzz/ZInAppPurchase)，或者直接在*CocoaPods*拉取*ZInAppPurchase*。（第一次试试上传到CocoaPods，还没加demo）



程序内购买要做的话要考虑很多，像漏单处理、重新购买处理等等，对于游戏App来说更需要考虑。下面只介绍最简单的流程和处理。



## 程序内购买流程

iOS程序内购买流程主要分几步：

1. iTunes Connect商品配置
2. 添加沙箱技术测试员
3. App内获取购买商品
4. 验证receipt

### iTunes Connect商品配置

主要是填写完整信息和添加商品。

#### 填写完整信息

登录*iTunes Connect*，进入”*协议、税务和银行业务*“。

如果*Contracts In Process*下有*All(See Contract)*和*Contact Info*、*Bank Info*、*Tax Info*三列，则表示已填写；否则点击*Request*按照提示进行操作。之后就会出现*Contact Info*、*Bank Info*、*Tax Info*三列，分别*Set Up*(需要同公司财务人员一起填写)。

(如果没有填写完整只能添加*免费订阅*商品)

#### 添加商品

登录*iTunes Connect*，进入*我的App*——*功能*——*App内购买项目*，点击+号。可以添加的类型有：消耗型项目、非消耗型项目、自动续订订阅、免费订阅、非续订订阅。商品添加完屏幕快照就会变成*准备提交*状态。

注意：产品 ID不可重复，如果删除某个商品，以后这个产品的ID也不可用，即使它已经被删除了；另外类型也不能改，选错了只能重新增加一个商品。


### 添加沙箱技术测试员

登录*iTunes Connect*，进入*用户和职能*——*沙箱技术测试员*，点击+号。（必须是未注册的Apple账号，用于测试购买）



### App内获取购买商品

- 导入系统库StoreKit

```
import StoreKit
```

- 获取商品信息

根据productId获取商品信息(可以获取多个)： 

```
let productRequest = SKProductsRequest(productIdentifiers: Set<String>(arrayLiteral: productId))
productRequest.delegate = self
productRequest.start()
```

实现SKProductsRequestDelegate：

```
func productsRequest(request: SKProductsRequest, didReceiveResponse response: SKProductsResponse) {
    if let product = response.products.first {// 获取返回的商品
    }
}
```

- 购买商品

购买获取的商品product：

```
if SKPaymentQueue.canMakePayments() {// 是否能且允许支付
    let payment = SKPayment(product: product)
    SKPaymentQueue.defaultQueue().addTransactionObserver(self)
    SKPaymentQueue.defaultQueue().addPayment(payment)
}
```

实现SKPaymentTransactionObserver：

```
func paymentQueue(queue: SKPaymentQueue, updatedTransactions transactions: [SKPaymentTransaction]) {

    for transaction in transactions {
        switch transaction.transactionState {
        case .Purchased: // Transaction is in queue, user has been charged.  Client should complete the transaction.

            if let receiptUrl = NSBundle.mainBundle().appStoreReceiptURL, let receiptData = NSData(contentsOfURL: receiptUrl) {
                let receiptString = receiptData.base64EncodedStringWithOptions(NSDataBase64EncodingOptions(rawValue: 0))
                // 将receiptString发给服务器
            }
            SKPaymentQueue.defaultQueue().finishTransaction(transaction)

        case .Failed: // Transaction was cancelled or failed before being added to the server queue.

            if let errorCode = transaction.error?.code {
            }
            SKPaymentQueue.defaultQueue().finishTransaction(transaction)
        default:
            break
        }
    }
}
```



### 验证receipt

receipt验证可以本地验证，也可以提交给App Store验证。

参考链接：[Validating Receipts With the App Store](https://developer.apple.com/library/mac/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html#//apple_ref/doc/uid/TP40010573-CH104-SW1)

我们是将receipt进行base64编码后，传给服务器，服务器判断凭证是否已经存在或验证过，再去POST给Apple服务器验证。

- 沙箱环境POST的URL

https://sandbox.itunes.apple.com/verifyReceipt

- 正式环境POST的URL

https://buy.itunes.apple.com/verifyReceipt

验证后Apple会返回数据，从中可以获取product_id、quantity等，下面是正确时的返回数据：

```
{
    "status": 0,
    "environment": "Sandbox",
    "receipt": {
        "receipt_type": "ProductionSandbox",
        "adam_id": 0,
        "app_item_id": 0,
        "bundle_id": "com.xxx.xxxxxx",
        "application_version": "999",
        "download_id": 0,
        "version_external_identifier": 0,
        "receipt_creation_date": "2016-05-26 04:35:08 Etc/GMT",
        "receipt_creation_date_ms": "1464237308000",
        "receipt_creation_date_pst": "2016-05-25 21:35:08 America/Los_Angeles",
        "request_date": "2016-05-26 06:40:32 Etc/GMT",
        "request_date_ms": "1464244832729",
        "request_date_pst": "2016-05-25 23:40:32 America/Los_Angeles",
        "original_purchase_date": "2013-08-01 07:00:00 Etc/GMT",
        "original_purchase_date_ms": "1375340400000",
        "original_purchase_date_pst": "2013-08-01 00:00:00 America/Los_Angeles",
        "original_application_version": "1.0",
        "in_app": [
            {
                "quantity": "1",
                "product_id": "000000",
                "transaction_id": "1000000213676495",
                "original_transaction_id": "1000000213676495",
                "purchase_date": "2016-05-26 04:35:08 Etc/GMT",
                "purchase_date_ms": "1464237308000",
                "purchase_date_pst": "2016-05-25 21:35:08 America/Los_Angeles",
                "original_purchase_date": "2016-05-26 04:35:08 Etc/GMT",
                "original_purchase_date_ms": "1464237308000",
                "original_purchase_date_pst": "2016-05-25 21:35:08 America/Los_Angeles",
                "is_trial_period": "false"
            }
        ]
    }
}
```



## 测试截图

下面是在沙箱环境下的真机测试截图（“测试”是所填写的产品名称，未登录Apple ID时会提示登录，已登录时会提示输入密码/Touch ID）：
![IMG_6816.PNG](304530-e6031ed92ab8fac0.png)

![IMG_6814.PNG](304530-8e271b6afd1bc2eb.png)



## 提交审核

在*iTunes Connect*添加完App版本后，在*App 内购买项目*处添加该版本新增的App内购买项目。




## 审核问题

####找不到入口

Information Needed

We have begun the review of your app but aren't able to continue because we can't locate the In-App Purchase(s) within your app. 

At your earliest opportunity, please reply to this message providing the steps for locating the In-App Purchase(s) in your app.


####使用In-App Purchase之前要求登录/注册

Hello, 

Thank you for providing the information. Upon further review of your application, we have found the following issue(s):

Legal - 5.1.1


We noticed that your app requires users to register with personal information to access non account-based features(including In-App Purchase feature), which is not allowed on the App Store. 

We've attached screenshot(s) for your reference.

Apps cannot require user registration prior to allowing access to app content and features that are not associated specifically to the user.

Next Steps

User registration that requires the sharing of personal information must be optional or tied to account-specific functionality.

Please make it clear to the user that registering will enable them to access the content from any of their iOS devices, and to provide them a way to register at any time, if they wish to later extend access to additional iOS devices.

Please note that although guideline 3.1.2 of the App Store Review Guidelines requires an application to make subscription content available to all the iOS devices owned by a single user, it is not appropriate to force user registration to meet this requirement; such user registration must be made optional.



-END-
欢迎到我的博客交流：https://zackzheng.info