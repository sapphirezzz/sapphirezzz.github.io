---
title: 通过iPhone设备的identifier精确判断设备型号和是否是全屏设备
date: 2018-02-07 16:00:00
tags: 
     - iOS
categories: iOS
keywords: iOS model swift identifier
description: 记录了iPhone设备的identifier，并以此判断iPhone设备的机型
---

记录 iPhone 设备的 identifier ，并以此判断设备型号。

```swift
import UIKit

extension UIDevice {

    public enum iPhoneDevice {

        enum DeviceIdentifier: String {

            case iPhone1_1 = "iPhone1,1"
            case iPhone1_2 = "iPhone1,2"
            case iPhone2_1 = "iPhone2,1"
            case iPhone3_1 = "iPhone3,1"
            case iPhone3_2 = "iPhone3,2"
            case iPhone3_3 = "iPhone3,3"
            case iPhone4_1 = "iPhone4,1"
            case iPhone5_1 = "iPhone5,1"
            case iPhone5_2 = "iPhone5,2"
            case iPhone5_3 = "iPhone5,3"
            case iPhone5_4 = "iPhone5,4"
            case iPhone6_1 = "iPhone6,1"
            case iPhone6_2 = "iPhone6,3"
            case iPhone7_1 = "iPhone7,1"
            case iPhone7_2 = "iPhone7,2"
            case iPhone8_1 = "iPhone8,1"
            case iPhone8_2 = "iPhone8,2"
            case iPhone8_4 = "iPhone8,4"
            case iPhone9_1 = "iPhone9,1"
            case iPhone9_2 = "iPhone9,2"
            case iPhone9_3 = "iPhone9,3"
            case iPhone9_4 = "iPhone9,4"
            case iPhone10_1 = "iPhone10,1"
            case iPhone10_2 = "iPhone10,2"
            case iPhone10_3 = "iPhone10,3"
            case iPhone10_4 = "iPhone10,4"
            case iPhone10_5 = "iPhone10,5"
            case iPhone10_6 = "iPhone10,6"
            case iPhone11_2 = "iPhone11,2"
            case iPhone11_4 = "iPhone11,4"
            case iPhone11_6 = "iPhone11,6"
            case iPhone11_8 = "iPhone11,8"
            case iPhone12_1 = "iPhone12,1"
            case iPhone12_3 = "iPhone12,3"
            case iPhone12_5 = "iPhone12,5"
        }

        case iPhone
        case iPhone3G
        case iPhone3GS
        case iPhone4
        case iPhone4S
        case iPhone5
        case iPhone5c
        case iPhone5s
        case iPhone6
        case iPhone6Plus
        case iPhone6s
        case iPhone6sPlus
        case iPhoneSE
        case iPhone7
        case iPhone7Plus
        case iPhone8
        case iPhone8Plus
        case iPhoneX
        case iPhoneXs
        case iPhoneXsMax
        case iPhoneXr
        case iPhone11
        case iPhone11Pro
        case iPhone11ProMax

        init(identifier: DeviceIdentifier) {
            switch identifier {
            case .iPhone1_1:
                self = .iPhone
            case .iPhone1_2:
                self = .iPhone3G
            case .iPhone2_1:
                self = .iPhone3GS
            case .iPhone3_1, .iPhone3_2, .iPhone3_3:
                self = .iPhone4
            case .iPhone4_1:
                self = .iPhone4S
            case .iPhone5_1, .iPhone5_2:
                self = .iPhone5
            case .iPhone5_3, .iPhone5_4:
                self = .iPhone5c
            case .iPhone6_1, .iPhone6_2:
                self = .iPhone5s
            case .iPhone7_2:
                self = .iPhone6
            case .iPhone7_1:
                self = .iPhone6Plus
            case .iPhone8_1:
                self = .iPhone6s
            case .iPhone8_2:
                self = .iPhone6sPlus
            case .iPhone8_4:
                self = .iPhoneSE
            case .iPhone9_1, .iPhone9_3:
                self = .iPhone7
            case .iPhone9_2, .iPhone9_4:
                self = .iPhone7Plus
            case .iPhone10_1, .iPhone10_4:
                self = .iPhone8
            case .iPhone10_2, .iPhone10_5:
                self = .iPhone8Plus
            case .iPhone10_3, .iPhone10_6:
                self = .iPhoneX
            case .iPhone11_2:
                self = .iPhoneXs
            case .iPhone11_4, .iPhone11_6:
                self = .iPhoneXsMax
            case .iPhone11_8:
                self = .iPhoneXr
            case .iPhone12_1:
                self = .iPhone11
            case .iPhone12_3:
                self = .iPhone11Pro
            case .iPhone12_5:
                self = .iPhone11ProMax
            }
        }
    }

    private var identifier: String {

        var systemInfo = utsname()
        uname(&systemInfo)
        let machineMirror = Mirror(reflecting: systemInfo.machine)
        let identifier = machineMirror.children.reduce("") { identifier, element in
            guard let value = element.value as? Int8, value != 0 else { return identifier }
            return identifier + String(UnicodeScalar(UInt8(value)))
        }
        return identifier
    }

    public var isPad: Bool {
        return userInterfaceIdiom == UIUserInterfaceIdiom.pad
    }

    public var isPhone: Bool {
        return userInterfaceIdiom == UIUserInterfaceIdiom.phone
    }

    public var iPhoneGeneration: iPhoneDevice? {

        let identifier = UIDevice.current.identifier
        guard let deviceIdentifier = iPhoneDevice.DeviceIdentifier(rawValue: identifier) else {
            return nil
        }
        return iPhoneDevice(identifier: deviceIdentifier)
    }

    public var isFullScreenSeries: Bool {
        #if DEBUG
            return ((UIScreen.main.bounds.width, UIScreen.main.bounds.height) == (375, 812) && UIScreen.main.scale == 3.0) || ((UIScreen.main.bounds.width, UIScreen.main.bounds.height) == (414, 896) && (UIScreen.main.scale == 3.0 || UIScreen.main.scale == 2.0))
        #else
        return [
            iPhoneDevice.iPhoneX,
            iPhoneDevice.iPhoneXs,
            iPhoneDevice.iPhoneXsMax,
            iPhoneDevice.iPhoneXr,
            iPhoneDevice.iPhone11,
            iPhoneDevice.iPhone11Pro,
            iPhoneDevice.iPhone11ProMax
        ].contains(iPhoneGeneration)
        #endif
    }
}
```

使用：

```swift
let model = UIDevice.current.iPhoneGeneration
let isFullScreenSeries = UIDevice.current.isFullScreenSeries
```



> 设备 identifier 摘自 [Model](https://www.theiphonewiki.com/wiki/Models) 



-END-
欢迎到我的博客交流：https://zackzheng.info