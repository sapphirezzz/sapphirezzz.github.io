---
title: 手把手教你深入玩转SwiftUI（1）
date: 2021-01-05 22:00:00
tags: 
     - SwiftUI
categories: Tech
description: 手把手教你深入玩转SwiftUI系列开篇

---

> `Swift`是世界上最好的语言！

# 一、前言
作为一个使用 Swift 已经六年的程序猿，SwiftUI 已经发布一年多了，最近才终于有时间实践了一番。决定写写这个系列，这是仅次于 Swift 的另一重要发布，下一代的客户端 UI 开发技术，刚好前段时间有个产品经理的朋友说想学。

除了分享基础知识，还会讲解深层次的**要点精髓**。**这才是学习的重点。**

从苹果的官方的教程开始。这教程非常给力，交互非常好，例子简练精要，可以感受到`SwiftUI`受到多大的重视。

> 本文中的例子来自文末附录带的苹果官方 Demo

# 二、声明式 UI 编程

相信很多写过 React 的原生同学会有个比较深的感受，界面开发起来很方便，那是因为，人家已经是“**声明式编程**”了。iOS 还在用着 UIKit 和 Storyboard 繁琐地写着描述式 UI 和设置约束。

同时 React 的*状态管理*和*数据绑定*也极大地刺激着我们。在Controller、View 和 Model 间来回转悠的我们早已疲乏，项目越来越需要花精力重构，增大维护成本。虽然也有些办法处理，但去不到根源。

不可否认 UIKit 曾经作出巨大贡献，降低了我们的学习曲线，但终究是要被替代的，就像 Swift 替代 OC，只是时间问题。

`SwiftUI`是新一代界面编程思想——声明式编程，本质是一套`DSL`。网上有很多介绍，在此就不赘述了。

# 三、课程拆解
官方教程这一篇分六个 Section，主要讲解几个点：
1. 创建 SwiftUI 项目
2. 使用 SwiftUI 预览功能
3. 自定义组件
4. 使用 Text、Image、VStack、HStack、Spacer 等 UI 组件
5. 使用其他库的组件

下面主要从一个个文件的角度来拆解要点和各种新特性。

## 创建 SwiftUI 项目

- 打开 Xcode -> File -> New -> Project...
- 弹窗 App
-> Next
 -> 填写 Product Name（项目名），如 LandmarksApp
-> Interface 选择 SwiftUI
-> Life Cycle 选择 SwiftUI App
-> 选择存放的目录 Create

## LandmarksApp.swift

```
import SwiftUI

@main
struct LandmarksApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

- SwiftUI
这里`import`了一个新的库`SwiftUI`，包含以前常用的 UI 库，并新定义了一些 UI 相关的类。其中最重要的是`Combine`，后面会简单说到。
```
import Combine
import CoreData
import CoreFoundation
import CoreGraphics
import Darwin
import DeveloperToolsSupport
import Foundation
import SwiftUI
import UIKit
import UniformTypeIdentifiers
import os.log
import os
import os.signpost
```
- View

SwiftUI 库中定义了`View`，可以看到是个协议，定义了屏幕上一个元素符合的条件。自定义新的 UI 类只要遵循这个协议，就可以被展示。这种设计比以前的`UIView`作为 UI 父类更轻便。

而且它的`body`属性本质也是`View`，这种设计是`SwiftUI`的精髓，下面会讲到。
```
public protocol View {

    /// The type of view representing the body of this view.
    ///
    /// When you create a custom view, Swift infers this type from your
    /// implementation of the required `body` property.
    associatedtype Body : View

    /// The content and behavior of the view.
    @ViewBuilder var body: Self.Body { get }
}
```

- 修饰词`@main`

这里提供了程序的启动入口。

使用 SwiftUI 的应用的生命周期需要符合`App`协议，当前是定义了一个结构体类型，叫`SwiftUIDemoApp `，遵循了`App`协议。使用类遵循协议也可以，不一定要结构体。

- 关键字`some`

这里使用`some`返回了一个[Opaque Result Type](https://github.com/apple/swift-evolution/blob/master/proposals/0244-opaque-result-types.md)（反向泛型类型，也称不透明结果类型）。反向泛型类型的类型参数，是由实现指定并隐藏起来的，一般的泛型是由使用者指定的。
```
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello Zack !")
            Image("zack-avatar")
        }
    }
}
```
比如上面的代码，实际上返回的类型是`VStack<TupleView<(Text, Image)>>`，由系统自行推断。我们可以任意组合 UI 组件内部的实现（包括运行时的变动），不用去指明具体类型，节省了非常多的工作量。

Swift 泛型系统长期存在一个问题是，带有 associatedtype 的协议不能作为类型使用，只能作为类型约束使用，无法作为函数的返回参数。而`Some`的设计，使这个问题得到解决。

**因此可以看出，正是`View`的递归设计和`Some`的隐藏特性，支撑了`SwiftUI`的声明式以及高度可组合，这是它的精髓！**

## CircleImage.swift

```
import SwiftUI

struct CircleImage: View {
    var body: some View {
        Image("zackzheng")
            .clipShape(Circle())
            .overlay(Circle().stroke(Color.white, lineWidth: 4))
            .shadow(radius: 7)
    }
}

struct CircleImage_Previews: PreviewProvider {
    static var previews: some View {
        CircleImage()
    }
}
```
- Image

使用`Image`组件展示图片，只需在`Assets.xcassets`中添加素材，再用素材名称初始化即可，比起以前方便一点。

- 链式调用

这里定义了一个结构体`CircleImage `。`body`需要返回遵循`View`协议的类型，因此可以推测`.clipShape`、`.overlay`、`.shadow`几个链式调用上的方法返回的都是`View`，查看定义可验证。

这里几个方法从命名得知是形状修剪、覆盖边框、阴影等，不需赘述。

**`链式调用`的设计也为声明式提供了简化和灵活度，不用如 React 一样将所有属性都配置。**

- PreviewProvider

A type that produces view previews in Xcode.

这是用于实现`SwiftUI`在 Xcode 中的实时预览特性，在 Xcode 默认是界面右侧可以看到（点击右侧上方的 Resume，或者在菜单 Editor > Canvas 打开）。

Amazing！做过前端开发的原生同学，都羡慕解释型语言的这个特点，虽然之前 Swift Playground 已经可以实时运行结果，但这次是 UI 啊！对标 Weex 或 React Native 的 Hot Reloading。

这里只是支持 Xcode 预览，所以这部分代码并不是必须的。
每个`.swift`文件可以写多个，预览会纵向排列。另外`previews`属性实际上可以任意返回，即使不是你当前新自定义的类。

> 系统需要升级到 11.x macOS Big Sur 
> 使用搭载有 SwiftUI.framework 的 macOS 10.15 才能够看到预览

- 修改组件属性

在预览处处于 Preview 状态下，command + 点击组件，会弹出菜单，点击“Show SwiftUI Inspector”，就可以对组件属性进行修改。

## MapView.swift

```
import SwiftUI
import MapKit

struct MapView: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 34.011_286, longitude: -116.166_868),
        span: MKCoordinateSpan(latitudeDelta: 0.2, longitudeDelta: 0.2)
    )

    var body: some View {
        Map(coordinateRegion: $region)
    }
}

struct MapView_Previews: PreviewProvider {
    static var previews: some View {
        MapView()
    }
}
```

这里介绍了引用其他库的组件，MapKit 库中的`Map `组件。

- `@State`

@State 是属性修饰词。
> SwiftUI 将会把使用过`@State`修饰器的属性存储到一个特殊的内存区域，并且这个区域和 View struct 是隔离的. 当`@State`装饰过的属性发生了变化，SwiftUI 会根据新的属性值重新创建视图。

当我们需要属性变化时，界面就自动刷新，则需要给属性添加`@State`修饰，类似于Vue和React中的State。以后不需要通过`didSet`来实现类似功能。

- @Binding

在 Swift 中基础属性变量的传递形式是值类型传递，不是引用传递。通过在变量前面添加`$`，可以变成引用传递，如代码中的`$region`。这样当 region 变化时，由于`@State`的关系，界面会重新渲染，Map 组件就会接收到属性的新值。

**所以这两个装饰器结合使用，达到通过属性重新渲染子组件的目的。**

**监听和渲染底层是通过`Combine`实现，可以看到下面 View 的 extension，这里不展开讲。**

```
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
extension View {

    /// Adds an action to perform when this view detects data emitted by the
    /// given publisher.
    ///
    /// - Parameters:
    ///   - publisher: The publisher to subscribe to.
    ///   - action: The action to perform when an event is emitted by
    ///     `publisher`. The event emitted by publisher is passed as a
    ///     parameter to `action`.
    ///
    /// - Returns: A view that triggers `action` when `publisher` emits an
    ///   event.
    @inlinable public func onReceive<P>(_ publisher: P, perform action: @escaping (P.Output) -> Void) -> some View where P : Publisher, P.Failure == Never

}
```

## ContentView.swift

```
struct ContentView: View {
    var body: some View {
        VStack {
            MapView()
                .ignoresSafeArea(edges: .top)
                .frame(height: 300)

            CircleImage()
                .offset(y: -130)
                .padding(.bottom, -130)

            VStack(alignment: .leading) {
                Text("Turtle Rock")
                    .font(.title)

                HStack {
                    Text("Joshua Tree National Park")
                    Spacer()
                    Text("California")
                }
                .font(.subheadline)
                .foregroundColor(.secondary)

                Divider()

                Text("About Turtle Rock")
                    .font(.title2)
                Text("Descriptive text goes here.")
            }
            .padding()

            Spacer()
        }
    }
}
```
- VStack、HStack

这两个组件其实就是`UIStackView`的 SwiftUI 版本，`VStack `是子组件纵向排列，`HStack `是横向排列，都可以分别设置两个方向上的排列对齐方式。

相信用过 UIStackView 的同学都十分喜爱它，小编也用到爱不释手，并且逐渐替代了很多 UI 布局场景，广泛使用，放弃了麻烦的`NSLayoutConstraint`。

它们的初始化方式很有意思，放几个组件就可以了。
```
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
@frozen public struct VStack<Content> : View where Content : View {

    /// Creates an instance with the given spacing and horizontal alignment.
    ///
    /// - Parameters:
    ///   - alignment: The guide for aligning the subviews in this stack. This
    ///     guide has the same vertical screen coordinate for every child view.
    ///   - spacing: The distance between adjacent subviews, or `nil` if you
    ///     want the stack to choose a default distance for each pair of
    ///     subviews.
    ///   - content: A view builder that creates the content of this stack.
    @inlinable public init(alignment: HorizontalAlignment = .center, spacing: CGFloat? = nil, @ViewBuilder content: () -> Content)

    /// The type of view representing the body of this view.
    ///
    /// When you create a custom view, Swift infers this type from your
    /// implementation of the required `body` property.
    public typealias Body = Never
}
```
可以看到，实际上要求的是`Content`类型。有兴趣可以自行查阅[Funtion builders](https://github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md)了解，我就不说了。

**从 Demo 也可以看出，这两个组件会成为以后 SwiftUI 布局的基础组件。配合 offset 和 padding 成为主要的布局配合。**需要抛弃掉传统的依赖`NSLayoutConstraint`布局的惯性思维。所以小编适应起来挺快的。

- .ignoresSafeArea(edges: .top)

处于页面顶部安全区内的组件，通过`ignoresSafeArea`进行适配，具体就不作介绍。

- offset、padding

这两个方法可以改改 Demo ，就能看出区别了。
在 VStack、HStack 下，`offset`相当于改变`bounds`，不会改变`frame`。而`padding`相反，会改变`frame`，不会改变`bounds`。

- `Bool` 的 `toggle()`

今天还看到`Struct`类型`Bool`有个`mutating`方法`toggle`，作用就是将`true`变为`false`，或者将`false`变为`true`。

`mutating`大家应该都很熟悉了，就是`Struct`用于修改自身属性的方法定义。所以这个方法很实用，比以前写`A = !A`感觉自然多了。

> Swift 新手概念：生命周期、协议、结构体、类、范型、import、Safe Area(安全区) 、bounds、frame、mutating

# 四、总结

至此，官方教程的第一章第一节，我们讲解完了。后面会继续整理剩下的章节。

从上我们可以总结出几点：

1. 协议在`SwiftUI`的整体设计上发挥了巨大的作用
2. `SwiftUI`完美满足 UI 快速实时预览的需求
3. `@Binding`和`@State`实现了数据流层面的界面状态绑定
4. `some`实现的`Opaque Result Types`解决了范型存在已久的问题，为更好的设计方式提供了途径
5. `VStack`、`HStack`、`offset`、`padding`将构成`SwiftUI`布局的基础，替代繁琐的`NSLayoutConstraint`
6. 链式调用的设计简化了声明式的灵活实现
7. **`View`的递归设计和`some`的隐藏特性，支撑了`SwiftUI`的声明式以及高度可组合特性，这是它的精髓**
8. 声明式和数据绑定的特性，极大地提高我们的效率，降低犯错几率、减少维护成本。

总而言之，`SwiftUI`，非常优秀！

# 五、附录
> [SwiftUI Tutorials](https://developer.apple.com/tutorials/swiftui)
> [SwiftUI Tutorials/Creating and Combining Views](https://developer.apple.com/tutorials/swiftui/creating-and-combining-views)
> [官方 Demo —— Project files](https://docs-assets.developer.apple.com/published/0ae7c8698688ae31636dedfd79a6153a/5200/CreatingAndCombiningViews.zip)



![代码运行效果](demo-screen-shot.jpg)



-END-
欢迎到我的博客交流：[https://zackzheng.info](https://zackzheng.info)