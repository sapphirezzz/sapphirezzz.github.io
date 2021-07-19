---
title: 手把手教你编写Swift脚本——一条命令完成AppIcon更换需求
date: 2021-07-14 22:00:00
tags: 
     - Swift
     - script
     - appIcon
categories: Tech
description: 通过完成更换AppIcon需求的脚本讲解Swift脚本的编写

---

> `Swift`是世界上最好的语言！

# 背景

提倡培养自动化思维处理日常工作，提高效率，时间和精力聚焦到更关键的点上。

对于具有重复性或周期性的工作可以思考使用各种自动化方式去解决，如替换`App` `Icon`的工作。

# 脚本使用

目前实现的功能是：

1. 生成`AppIcon.appiconset`文件夹和`Contents.json`文件
2. 生成所有需要的尺寸素材
3. 提交`commit`

- 使用

执行`reduceAppIconAndPush.swift`文件，带上素材的相对路径

```shell
swift reduceAppIconAndPush.swift ../../icon-1024.png
```

> **因为会提交`commit`，所以本地最好不要有未提交的`commit`，不然会一并被`push`上远端**
>
> **`commit`只会包含`Contents.json`文件和素材文件，对本地其他未暂存的文件没有影响；已暂存的内容会一并被提交**

# 脚本具体实现

抽出自动化流程为

1. 创建`AppIcon.appiconset`文件夹
2. 创建`Contents.json`文件
3. 向`Contents.json`文件写入素材的组织结构
4. 批量压缩成对应的素材
5. 提交改动`commit`

相比使用`Shell`或者`Python`，使用`Swift`命令行模式实现，更易于编写和方便团队其他人维护。

## 数据结构

```swift
struct AppIconImageItem: Codable {
    let size: String
    let idiom: String
    let filename: String
    let scale: String
}

struct AppIconInfo: Codable {
    let version: Int
    let author: String
}

struct AppIcon: Codable {
    var images: [AppIconImageItem]
    let info: AppIconInfo
}
```

## 创建`AppIcon.appiconset`文件夹

```swift
func createAppIconSetFile(filePath: String, appIcon: AppIcon) -> Bool {
    let fileManager = FileManager.default
    do {
        if fileManager.fileExists(atPath: filePath) {
            try fileManager.removeItem(atPath: filePath)
        }
        try fileManager.createDirectory(atPath: filePath, withIntermediateDirectories: true, attributes: nil)
        print("\(filePath) 文件夹创建成功")
        return true
    }catch {
        print("【失败】\(filePath) 文件夹创建失败")
        print(error.localizedDescription)
        return false
    }
}
```

## 创建`Contents.json`文件的素材组织结构

```swift
func createAppIconModel() -> AppIcon {
    let images: [AppIconImageItem] = [
        AppIconImageItem(size: "20x20", idiom: "iphone", filename: "icon-20@2x.png", scale: "2x"),
        AppIconImageItem(size: "20x20", idiom: "iphone", filename: "icon-20@3x.png", scale: "3x"),

        AppIconImageItem(size: "29x29", idiom: "iphone", filename: "icon-29.png", scale: "1x"),
        AppIconImageItem(size: "29x29", idiom: "iphone", filename: "icon-29@2x.png", scale: "2x"),
        AppIconImageItem(size: "29x29", idiom: "iphone", filename: "icon-29@3x.png", scale: "3x"),
        
        AppIconImageItem(size: "40x40", idiom: "iphone", filename: "icon-40@2x.png", scale: "2x"),
        AppIconImageItem(size: "40x40", idiom: "iphone", filename: "icon-40@3x.png", scale: "3x"),
        
        AppIconImageItem(size: "60x60", idiom: "iphone", filename: "icon-60@2x.png", scale: "2x"),
        AppIconImageItem(size: "60x60", idiom: "iphone", filename: "icon-60@3x.png", scale: "3x"),
        
        AppIconImageItem(size: "20x20", idiom: "ipad", filename: "icon-20-ipad.png", scale: "1x"),
        AppIconImageItem(size: "20x20", idiom: "ipad", filename: "icon-20@2x-ipad.png", scale: "2x"),
        
        AppIconImageItem(size: "29x29", idiom: "ipad", filename: "icon-29-ipad.png", scale: "1x"),
        AppIconImageItem(size: "29x29", idiom: "ipad", filename: "icon-29@2x-ipad.png", scale: "2x"),
        
        AppIconImageItem(size: "40x40", idiom: "ipad", filename: "icon-40.png", scale: "1x"),
        AppIconImageItem(size: "40x40", idiom: "ipad", filename: "icon-40@2x.png", scale: "2x"),
        
        AppIconImageItem(size: "76x76", idiom: "ipad", filename: "icon-76.png", scale: "1x"),
        AppIconImageItem(size: "76x76", idiom: "ipad", filename: "icon-76@2x.png", scale: "2x"),
        
        AppIconImageItem(size: "83.5x83.5", idiom: "ipad", filename: "icon-83.5@2x.png", scale: "2x"),
        
        AppIconImageItem(size: "1024x1024", idiom: "ios-marketing", filename: "icon-1024.png", scale: "1x"),
    ]

    let info = AppIconInfo(version: 1, author: "zhengzuanzhe")
    let appIcon = AppIcon(images: images, info: info)
    return appIcon
}
```

## 创建`Contents.json`文件并写入素材组织结构

```swift
func createContentsJsonFile(filePath: String, appIcon: AppIcon) -> Bool {

    let encoder = JSONEncoder()
    do {
        encoder.outputFormatting = .prettyPrinted
        let appIconData = try encoder.encode(appIcon)
        if let appIconStr = String(data: appIconData, encoding: .utf8) {
            let contentJsonUrl = URL(fileURLWithPath: filePath)
            try appIconStr.write(to: contentJsonUrl, atomically: true, encoding: .utf8)
            print("\(filePath) 文件创建成功")
            return true
        } else {
            print("【失败】\(filePath) 文件创建失败")
            return false
        }
    } catch {
        print(error.localizedDescription)
        return false
    }
}
```

## 读取素材文件

```swift
func cgImageWithPath(_ path: String) -> CGImage? {

    let url = URL(fileURLWithPath: path)
    guard let data = try? Data(contentsOf: url), let dataProvider = CGDataProvider(data: data as CFData), let image = CGImage(pngDataProviderSource: dataProvider, decode: nil, shouldInterpolate: true, intent: .defaultIntent) else {
        return nil
    }
    return image
}
```

## 批量压缩成对应的素材

```swift
func createImage(size: CGSize, scale: CGFloat, image: CGImage, filename: String) {
    let width  = Int(size.width * scale)
    let height = Int(size.height * scale)
    let bitsPerComponent = image.bitsPerComponent
    let bytesPerRow = image.bytesPerRow
    let colorSpace  = image.colorSpace

    if let context = CGContext(data: nil,
                               width: width,
                               height: height,
                               bitsPerComponent: bitsPerComponent,
                               bytesPerRow: bytesPerRow,
                               space: colorSpace!,
                               bitmapInfo: CGImageAlphaInfo.premultipliedLast.rawValue) {
        context.interpolationQuality = .high
        context.draw(image, in: CGRect(origin: .zero, size: CGSize(width: width, height: height)))
        if let inputImage = context.makeImage() {
            let outputImagePath = "\(filename)"
            let outputUrl = URL(fileURLWithPath: outputImagePath) as CFURL
            let destination = CGImageDestinationCreateWithURL(outputUrl, kUTTypePNG, 1, nil)
            if let destination = destination {
                CGImageDestinationAddImage(destination, inputImage, nil)
                if CGImageDestinationFinalize(destination) {
                    print("图片 \(filename) 生成成功")
                }else {
                    print("【失败】图片 \(filename) 生成失败")
                }
            }
        }else {
            print("【失败】图片 \(filename) 生成失败")
        }
    }
}
```

## 使用`Shell`提交`commit`

```swift
@discardableResult
func execute(path: String, arguments: [String]? = nil) -> Int {
    let process = Process()
    process.launchPath = path
    if arguments != nil {
        process.arguments = arguments!
    }
    process.launch()
    process.waitUntilExit()
    return Int(process.terminationStatus)
}

execute(path: "/usr/bin/git", arguments: ["add", "\(contentJsonPath)"])
appIcon.images.forEach({ imageItem in
    let filename = "\(appIconSetFilePath)/\(imageItem.filename)"
    execute(path: "/usr/bin/git", arguments: ["add", "\(filename)"])
})
execute(path: "/usr/bin/git", arguments: ["commit", "-m", "chore: 替换AppIcon"])
execute(path: "/usr/bin/git", arguments: ["push", "origin"])
```