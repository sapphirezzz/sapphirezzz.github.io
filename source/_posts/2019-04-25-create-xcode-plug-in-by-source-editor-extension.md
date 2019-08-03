---
title: 手把手教你用Source Editor Extension开发Xcode插件
date: 2019-04-24 22:00:00
tags: 
     - Xcode插件
     - iOS
categories: iOS
keywords: Xcode插件 SourceEditorExtension
description: 详细介绍了如何使用Xcode Source Editor Extension 开发 Xcode 插件，以及实现了一个将代码的 import 排序的 Demo
---

# 目录

+ 前言
+ Xcode 插件史
+ 实现插件
  - 创建 macOS 应用
  - 编写插件代码
  - 修改插件命名
  - 调试插件
  - 分发插件
  - 设置快捷键
+ 进阶讲解
  - Demo 逻辑
  - Plist 文件处理
  - XCSourceEditorCommand 协议
  - XCSourceEditorExtension 协议
+ 总结



# 前言

一个项目工程，随着架构的阶段性稳定、公共组件的抽离和代码规范的制定等，势必会进入一个"重复劳动"的阶段。所谓"重复劳动"，即需求都有固定的模式和分解步骤去完成，大部分是重复、一致的代码编写，只有少部分工作需要思考、抽象、实现。但往往这些"重复劳动"占据了大部分时间成本，而且由于其机械性所以最容易出现问题。

于是将重复劳动自动化，即用代码写代码，是一个团队的重点工作之一，让成员将时间和精力放在更值得关注的事情上。而开发 IDE 插件，可以实现这种代码层面的自动化。

本来想做个插件，实现生成 cell 的 .xib 和 .swift 文件并自动关联等功能，练练手，结果发现目前 Xcode 开放的插件并不能支持。



# Xcode插件史

在Xcode 8之前，Xcode 插件有着比较辉煌的发展，各种便利的插件、专门的插件管理工具 Alcatraz 等。

但从 Xcode 8 开始，出于安全性考虑（比如说 Xcode ghost 事件），Apple 不再支持第三方的插件，但提供了解决方案—— Xcode Source Editor Extension，目前只能完成有限的文本编辑辅助。

> 本文Demo：[https://github.com/sapphirezzz/ZXcodeExtension](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsapphirezzz%2FZXcodeExtension)）



# 实现插件

本文开发环境：Xcode Version 10.2.1 (10E1001)

## 创建macOS应用

打开Xcode，File->New->Project…，选择 macOS->Application>Cocoa App，填写 Product Name

![20190424135311](20190424135311.jpg)

新建 Target，File->New->Target…，选择 macOS->Application Extension->Xcode Source Editor Extension，填写 Product Name，如 ZExtension。在弹窗中选择 Activate。

**注意：该 Target 的命名会成为后面使用插件时一级菜单名称。**

![20190424135633](20190424135633.jpg)

## 编写插件代码

修改 SourceEditorCommand.swift 文件。

> 以下代码实现将import排序的功能

```swift
func perform(with invocation: XCSourceEditorCommandInvocation, completionHandler: @escaping (Error?) -> Void ) -> Void {
    // Implement your command here, invoking the completion handler when done. Pass it nil on success, and an NSError on failure.

    let linesToSort = invocation.buffer.lines.filter { line in
        return (line as? String)?.hasPrefix("import") ?? false
    }

    guard linesToSort.count > 0 else {
        completionHandler(nil)
        return
    }

    let firstLineIndex = invocation.buffer.lines.index(of: linesToSort[0]) // For insert
    guard firstLineIndex >= 0 else {
        completionHandler(nil)
        return
    }
    invocation.buffer.lines.removeObjects(in: linesToSort)
    let linesSorted = (linesToSort as? [String] ?? []).sorted() {$0 <= $1}
    linesSorted.reversed().forEach { (line) in
        invocation.buffer.lines.insert(line, at: firstLineIndex)
    }
    let selectionsUpdated: [XCSourceTextRange] = (0..<linesSorted.count).map { (index) in
        let lineIndex = firstLineIndex + index
        let endColumn = linesSorted[index].count - 1
        return XCSourceTextRange(start: XCSourceTextPosition(line: lineIndex, column: 0), end: XCSourceTextPosition(line: lineIndex, column: endColumn))
    }
    invocation.buffer.selections.setArray(selectionsUpdated)
    completionHandler(nil)
}
```

## 修改插件命名

在 ZExtension/Info.plist 中可以修改插件名称，对应的 Key 是 XCSourceEditorCommandName，支持中文。

不修改则默认是 Source Editor Command。

## 调试插件

选择该新建的 Scheme，如 ZExtension，运行（Command+R）。在弹窗中选择 Xcode，点 击Run。

![20190424145603](20190424145603.jpg)

接下来会弹出灰色的Xcode界面，新建项目或者打开测试项目。本文用了测试项目 Test。

![20190424150404](20190424150404.jpg)

使用插件排序，点击 Editor->ZExtension->Source Editor Command。

![20190424150422](20190424150422.jpg)

以下为插件运行后的结果：

![20190424150437](20190424150437.jpg)

结果显示，所有 import 排好序了！

## 分发插件

插件完成后，就需要投入工作中使用。

+ 上架 Mac App Store

编写的插件可以发布，上架到 Mac App Store。在 Xcode->Xcode Extensions… 可以看到上架的插件。笔者还没有发布，先略过。

+ 内部使用

在插件项目中，将 Products->ZXcodeExtension.app 文件拷贝到应用程序，并双击打开。此时在系统偏好设置->扩展->Xcode Source Editor，可以看到该插件，并且已勾选。重启 Xcode 就可以使用了。

## 设置快捷键

可以给插件设置快捷键，方便使用。

在 Xcode->Preferences…->Key Bindings->Editor Menu for Source Code，找到并设置。建议用 alt 如 alt+s，避免和其他快捷键冲突。



# 进阶讲解


实现之后，简单讲解下一些细节。

## Demo逻辑

Demo中主要操作了两个内容：

1. invocation.buffer.lines
2. invocation.buffer.selections

> lines 是当前编辑文件的每一行的内容，selections 是当前编辑文件选中的内容。

Demo 逻辑是：

1. 筛选出符合条件的行 linesToSort（以 import 开头）
2. 记录第一个符合条件的行的行数firstLineIndex，作为排序后的插入位置
3. 从 invocation.buffer.lines 中删除符合条件的行
4. 将符合条件的行进行排序得出 linesSorted
5. 将排序后的行插入 invocation.buffer.lines
6. 获取所有改动行信息 selectionsUpdated，设置 invocation.buffer.selections

主要是对 XCSourceEditorCommand 协议的实现。

## Plist 文件处理

Info.plist 文件中重要的 key 是 NSExtension 的 NSExtensionAttributes，包含两个 key：

1. XCSourceEditorCommandDefinitions
2. XCSourceEditorExtensionPrincipalClass

### XCSourceEditorCommandDefinitions

XCSourceEditorCommandDefinitions 是设置了每个命令（二级菜单）的信息：

1. XCSourceEditorCommandClassName
2. XCSourceEditorCommandIdentifier
3. XCSourceEditorCommandName

第一个是处理这个命令的类名，该类需实现 XCSourceEditorCommand 协议；第二个是每个命令的标示，用于 XCSourceEditorCommand 协议的方法区分处理命令；第三个是命令的展示名字。

### XCSourceEditorExtensionPrincipalClass

该扩展的类名，该类需实现 XCSourceEditorExtension 协议。

## XCSourceEditorCommand协议

```swift
/** A command provided by a source editor extension. There does not need to be a one-to-one mapping between command classes and commands: Multiple commands can be handled by a single class, by checking their invocation's commandIdentifier at runtime. */
@protocol XCSourceEditorCommand <NSObject>
```

根据官方注释，一个实现了 XCSourceEditorCommand 的类可以处理多种命令，即多个二级菜单，通过 invocation.commandIdentifier 来区分。而 commandIdentifier 是 Info.plist 中，XCSourceEditorCommandDefinitions 里面每一项的 XCSourceEditorCommandIdentifier 所定义的。

```swift
/** Perform the action associated with the command using the information in \a invocation. Xcode will pass the code a completion handler that it must invoke to finish performing the command, passing nil on success or an error on failure.
 
 A canceled command must still call the completion handler, passing nil.
 
 \note Make no assumptions about the thread or queue on which this method will be invoked.
 */
- (void)performCommandWithInvocation:(XCSourceEditorCommandInvocation *)invocation completionHandler:(void (^)(NSError * _Nullable nilOrError))completionHandler;
```

这是 XCSourceEditorCommand 协议定义的方法。

+ XCSourceEditorCommandInvocation

commandIdentifier 属性，用于区分不同命令；buffer，XCSourceTextBuffer 类型，主要用它的 lines 和 selections 属性。

+ completionHandler

实现逻辑之后，必须调用 completionHandler 以结束插件命令，成功时传参 nil，失败时传参 error 对象。即使取消处理也需要调用并传参 nil。

**结合 Plist 文件和 XCSourceEditorCommand 协议，我们可以编写处理多个命令的插件。**

## XCSourceEditorExtension协议

```swift
/** Invoked when the extension has been launched, which may be some time before the extension actually receives a command (if ever).
 
 \note Make no assumptions about the thread or queue on which this method will be invoked.
 */
- (void)extensionDidFinishLaunching;
```


插件被加载后的处理。



# 总结

可以看出，目前 Xcode Source Editor Extension 解决方案能实现的插件功能很有限，不支持UI交互，只能局限于文本处理上。希望以后苹果能扩展更多 API 供开发者使用。



-END-
欢迎到我的博客交流：https://zackzheng.info