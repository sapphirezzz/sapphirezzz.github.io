---
title: 手把手教你实现视频列表滚动自动播放-短视频列表滚动播放实战
date: 2019-09-22 15:30:00
tags: 
     - iOS
categories: iOS
keywords: iOS 短视频 滚动播放 滚动自动播放
description: 简单介绍了包含视频的列表滚动自动播放的方案
---

# 目录

- 前言
- 方案实现
  - 原始需求
  - 隐藏需求
  - 方案制定
  - 具体实现
- 总结
- 附录

# 前言

互联网内容已经逐渐从图文阅读往如今火热的短视频更迭，某种程度上短视频有着图文所没有的优势和不可替代性，降低了自我表达的门槛。近期迭代做了个短视频列表滚动自动播放的需求，上线了一段时间。觉得略有趣，简单分享下方案。

本文提供了 Demo ，将方案进行简化处理，只包含核心的功能实现。

> 本文的 Demo 附在文末

# 方案实现

![demo.gif](demo.gif)

介绍下整个方案的思考和实现的一些过程。

## 原始需求

简化如下：

- 自动播放条件

Wifi环境下，当视频中心位置从下往上越过屏幕的2/3位置，或从上往下越过屏幕的1/3位置时，视频开始自动播放；

- 停止播放条件

当视频中心位置离开可见区域时，视频自动停止播放。若下一个视频满足开始自动播放的条件，则上一个视频自动停止播放。

- 手动播放

点播放按钮可手动开始播放。开始播放某视频后，其他视频停止播放。

## 隐藏需求

原始需求背后需要考虑的其他情况：

- 页面进入/离开的处理
- App进入后台/返回前台的处理
- 从一个视频列表跳转到另一个视频列表的处理
- 页面包含安全区的处理
- 列表滚动播放的性能问题
- 视频循环播放/静音功能
- 上拉加载更多/下拉刷新等动作触发的滚动的处理
- 列表 cell 的复用问题

## 方案制定

主要考虑了以下一些方面：

1. 优先考虑的是性能问题，同一个列表中尽量只有一个视频控件
2. 尽量降低方案的侵入性，目前有多个现有列表需要支持该功能，涉及到多个 controller 和多个 cell ，侵入性低也利于改动和维护
3. 隐藏需求会影响到自动播放逻辑的调用时机

大致思路如下：

1. 由一个类管理整个 App 的视频滚动播放的相关逻辑，包括视频组件。
2. 由 controller 监控滚动，触发管理类进行处理。管理类计算当前符合自动播放的视频，播放并将视频组件嵌入 cell 中。
3. 为了降低侵入性，采用协议实现管理类和 controller、管理类和 cell 间的通信。也利于改动逻辑时只改动到管理类，而不是牵涉各个调用处和 cell。

隐藏需求处理：

1. 页面进入/离开的处理、从一个视频列表跳转到另一个视频列表的处理

在 controller 的生命周期中去调用管理类方法进行处理。离开时停止当前播放中的视频，进入时播放当前列表的视频。进入时判断数据源是否已经获取，已经获取则调用播放。另外在数据源获取后判断是否已经 Appear，是则调用播放。用变量标志避免两个逻辑重复调用。

2. App进入后台/返回前台的处理

App 进入后台/返回前台不会调用 controller 的生命周期，需要另外处理。

3. 页面包含安全区的处理

在计算视频的相对位置时，将安全区考虑在内进行计算。

4. 列表滚动播放的性能问题

测试**列表滚动停止时**、**实时滚动时**、**滚动降速到一定速度时**等情况下，调用自动播放逻辑的性能和体验，

5. 视频循环播放/静音功能

视频播放完毕的事件需要通过监听 NSNotification.Name.AVPlayerItemDidPlayToEndTime 实现，放在管理类中进行处理。

6. 上拉加载更多/下拉刷新等动作触发的滚动的处理

在这两种情况下，需要停止页面的逻辑调用，直到数据源返回成功或失败为止。

7. 列表 cell 的复用问题

复用时需要重新布置frame、清空数据源等。

## 具体实现

1. **定义管理类 VideoListAutoPlayManager**


```swift
class VideoListAutoPlayManager {
    
    private init() {
        playerVC.player = AVPlayer()
        playerVC.showsPlaybackControls = false
        playerVC.view.backgroundColor = UIColor.clear
    }
    static let shared = VideoListAutoPlayManager()
    
    private var playerVC: AVPlayerViewController = AVPlayerViewController()
    private var preOffsetY: CGFloat = 0
    private var currentPlayingView: VideoPlayable?
}
```

需要保存一些信息和状态，所以定义成单例。

AVPlayerViewController 自带控制条，需要隐藏。

视频播放时背景从黑色开始，会导致出现先看到封面，然后黑色，然后再播放视频的问题，设置为透明会让从封面到视频的过渡自然。

preOffsetY 记录当前滚动的 UIScrollView 的 contentOffset.y 。用于在多个视频满足自动播放时，通过判断滚动方向来决定选取哪个视频自动播放。

currentPlayingView 记录当前播放中的 cell。用于通知上一个播放的 cell 即将停止播放视频，方便 cell 处理另外的逻辑。

2. **定义协议**

```swift
protocol VideoPlayable: UIView {
    var viewToContainVideo: UIView {get}
    var urlToPlay: URL? {get}
    func videoStatusChanged(changeTo isPlaying: Bool)
}

protocol VideoListPlayable: UIScrollView {
    var visibleViews: [VideoPlayable] {get}
}

extension UITableView: VideoListPlayable {
    var visibleViews: [VideoPlayable] {
        let views: [VideoPlayable] = visibleCells.compactMap({ $0 as? VideoPlayable })
        return views
    }
}
extension UICollectionView: VideoListPlayable {
    var visibleViews: [VideoPlayable] {
        let views: [VideoPlayable] = visibleCells.compactMap({ $0 as? VideoPlayable })
        return views
    }
}
```

第一个协议，VideoPlayable，是存放视频的 cell 需要实现的。实现协议返回需要包含视频的 view ，需要播放的视频 URL，以及用于 VideoListAutoPlayManager 通知 cell 处理视频播放状态变化的调用方法。

第二个协议，VideoListPlayable，是滚动列表需要实现的。实现协议返回滚动列表当前可见的 cell，用于 VideoListPlayable 去判断哪些视频需要自动播放。

两个协议都遵循某个类，UIView 或 UIScrollView，是有些取巧，方便后面取 frame 等。也可以不遵循，然后在协议中返回需要的数据即可。

另外为 UITableView 和 UICollectionView 做了默认实现。

3. **触发滚动播放的处理**

```swift
func scrollViewDidScroll(_ scrollView: VideoListPlayable) {

    let currentOffsetY = scrollView.contentOffset.y
    let minY = scrollView.frame.height / 3
    let maxY = minY * 2
    // 获取在 scrollView 自动播放区域内的视频
    let autoPlayableViews = scrollView.visibleViews.filter { view in
        guard let relativeRect = relativeRect(view: view.viewToContainVideo, relativeTo: scrollView), view.urlToPlay != nil else {return false}
        let containerCenterY = relativeRect.minY + relativeRect.height / 2
        return (containerCenterY > minY && containerCenterY < maxY)
    }

    guard let first = autoPlayableViews.first else {
        // 没有需要自动播放的视频
        // 移除当前正在离开/已经离开屏幕的视频
        removeCurrentVideoIfLeavingScreen(scrollView: scrollView)
        preOffsetY = currentOffsetY
        return
    }

    // 取出需要自动播放的视频
    let viewToPlay: VideoPlayable = autoPlayableViews.reduce(first) { (result, view) in
        let isScrollToUpper = currentOffsetY < preOffsetY
        return result.frame.maxY > view.frame.maxY ? (isScrollToUpper ? view : result) : (isScrollToUpper ? result : view)
    }
    if let currentPlayingView = currentPlayingView, viewToPlay as UIView == currentPlayingView {
        // 满足条件的视频正在播放中
        preOffsetY = currentOffsetY
        return
    }
    removeCurrentVideo(on: scrollView)

    addPlayerView(to: viewToPlay, on: scrollView)
    preOffsetY = currentOffsetY
}
```

VideoListAutoPlayManager 提供该方法用于 controller 需要进行视频自动播放处理时进行调用。

> 外部可以自行决定在什么时机，进行视频自动播放逻辑的触发，不需要是在 scrollViewDidScroll 的时机。

该方法主要逻辑是：

取出当前可见区域中，满足自动播放条件（func relativeRect(view: UIView, relativeTo scrollView: VideoListPlayable) -> CGRect?）的 cell，即相对位置为滚动列表的 1/3 至 2/3 的位置。

如果没有满足条件的，则判断当前是否有播放中的视频，且视频即将或已经离开屏幕，有则停止播放视频，并通知 cell。

如果有满足条件的视频，则根据滚动方向选取视频（列表向上滚动时，播放靠下的视频，反之则播放靠上的视频），移除上一个播放中的视频（通知对应的 cell），切换视频源并播放，通知最新播放的 cell。

4. **手动播放的处理**

```swift
func play(at videoView: VideoPlayable, on scrollView: VideoListPlayable) {
    removeCurrentVideo(on: scrollView)
    addPlayerView(to: videoView, on: scrollView)
}
```

即移除当前播放中的视频，并将当前手动指定播放的视频进行播放。

5. **添加视频组件**

```swift
func addPlayerView(to view: VideoPlayable, on scrollView: VideoListPlayable) {

    guard let url = view.urlToPlay else {
        return
    }

    let avItem = AVPlayerItem(url: url)
    let avPlayer = AVPlayer(playerItem: avItem)
    playerVC.player = avPlayer
    avPlayer.isMuted = true
    avPlayer.play()

    view.videoStatusChanged(changeTo: true)

    let containerView = view.viewToContainVideo
    containerView.addSubview(playerVC.view)

    playerVC.view.translatesAutoresizingMaskIntoConstraints = false
    playerVC.view.topAnchor.constraint(equalTo: containerView.topAnchor).isActive = true
    playerVC.view.bottomAnchor.constraint(equalTo: containerView.bottomAnchor).isActive = true
    playerVC.view.leftAnchor.constraint(equalTo: containerView.leftAnchor).isActive = true
    playerVC.view.rightAnchor.constraint(equalTo: containerView.rightAnchor).isActive = true

    currentPlayingView = view
}
```

通过协议获取包含视频的 view，将视频放入其中，通知 cell 进行状态变化处理。

6. 隐藏需求的实现

基本按照上一个部分讲的思路实现，没有将这部分代码放到 Demo 中。

# 总结

目前的方案，cell只需实现协议，添加一个用于包含视频的 view 即可。这样降低了对原代码的侵入性、减少修改和维护的成本，可随时去除该自动播放的特性。另外隐藏需求实际花费的思考和时间会比原始需求多些，需要考虑很多细节。

# 附录

本文Demo：[VideoListPlayDemo](https://github.com/sapphirezzz/VideoListPlayDemo)



-END-
欢迎到我的博客交流：[https://zackzheng.info](https://links.jianshu.com/go?to=https%3A%2F%2Fzackzheng.info)
