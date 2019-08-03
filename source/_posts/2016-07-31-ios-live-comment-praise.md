---
title: iOS直播：评论框与粒子系统点赞动画
date: 2016-07-31 16:00:00
tags: 
     - iOS
     - 直播
categories: iOS
keywords: iOS 直播
description: 主要介绍了iOS直播的评论框、粒子系统点赞动画的实现
---

# 目录
- 前言
- 效果预览
- 评论框
  - 列表
  - 添加评论
  - 从下往上显示
  - 支持昵称颜色
  - 给出NSAttributedString
- 点赞动画

# 前言

最近做了直播功能，其实难度不是说很大，主要是方案和SDK的选择、整个直播流程的异常处理和优化，还有第三方SDK的填坑。不过本文只是记录下评论框和点赞效果的实现，其他的是用第三方SDK，觉得没什么好分享的，只是了解了直播流程和开发中会遇到的问题。
但看到效果还是蛮激动和蛮有成就感的，这个主要是技术本身带来的。

# 效果预览

![2016-07-31-ios-live-comment-praise-1.gif](304530-9c6fd4a095528f98.gif)

# 评论框

细化需求：

1. 显示评论内容
2. 从下往上显示
3. 最大支持1000条
4. 不同人昵称显示颜色随机分配，同一个人颜色保持不变。
5. 评论插入有动画

## 列表

- 新的类`MessageChatView`，对外接口`add`。

```
func add(message: String) {}
```

- 存放评论数组

```
private let maxMessageCount: Int = 1000
private var messages: [String] = []
```

- UITableViewDelegate & UITableViewDataSource

```
extension MessageChatView: UITableViewDataSource {
    func numberOfSectionsInTableView(tableView: UITableView) -> Int {
        return messages.count
    }
    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 1
    }
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {   
        ...
        return cell
    }
}
extension MessageChatView: UITableViewDelegate {
    func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
        return ...
    }
    func tableView(tableView: UITableView, heightForFooterInSection section: Int) -> CGFloat {
        return 5.0
    }
}
```

此时显示了数组里面的评论，最多1000条。

## 添加评论

```
func add(message: String) {
    messages.insert(message, atIndex: 0)
    tableView.insertSections(NSIndexSet(index: 0), withRowAnimation: .Top)
    if messages.count > maxMessageCount {
        messages.removeLast()
        tableView.deleteSections(NSIndexSet(index: messages.count), withRowAnimation: .None)
    }
}
```

使用`UITableView`自带的方法可以有动画效果。插入动画使用`.Top`。

## 从下往上显示

- `iOS`在tableView和tableViewCell里调用下面语句：

```
tableView.transform = CGAffineTransformMakeScale (1,-1);
```

```
label.transform = CGAffineTransformMakeScale (1,-1);
```

两条语句就可以实现了。

- `Android`可以调用ListView自带的属性`stackFromBottom`：

```
android:stackFromBottom="true"
```

> 网上有文章将数据 `append`到数据源，在获取数据源时从后往前读的方式(即messages.count-1-indexPath.section)，显然插入在0位置比那样更方便：`insert(message, atIndex: 0)`

## 支持昵称颜色

- 使用NSAttributedString，且由外界设置。messages类型改为NSAttributedString数组。

```
private var messages: [NSAttributedString] = []
```

- `add`改为NSAttributedString。

```
func add(message: NSAttributedString) {}
```

- 设置Label的时候设置label.attributedText。

## 给出NSAttributedString

- 一个新的类ChatColorText，对外接口colorText，参数nickName、text。

```
func colorText(nickName: String?, text: String?) -> NSAttributedString?{}
```

- 随机颜色数组。


```
private var colors = [
    UIColor(hex: .RGB00AEFF)!,
    UIColor(hex: .RGB00A61C)!,
    UIColor(hex: .RGB5400E6)!,
    UIColor(hex: .RGBFF3377)!,
    UIColor(hex: .RGBFF8800)!,
    UIColor(hex: .RGBFF5E00)!,
    UIColor(hex: .RGBCA2EE6)!,
]
```


- 记录当前取颜色的`Index`，使得不同人给不同颜色。 


```
private var colorIndex: Int = 0
```

- 记录昵称对应的颜色值，保证同一个昵称同一种颜色。

```
private var dicOfNameAndColor = [String: UIColor]()
```

- 对外接口colorText实现。

```
func colorText(nickName: String?, text: String?) -> NSAttributedString? {
    guard let nickName = nickName, text = text else {return nil}
    let nickNameColor: UIColor = {
        if let color = dicOfNameAndColor[nickName] {
            return color
        }else {
            let color = colors[colorIndex]
            dicOfNameAndColor[nickName] = color
            colorIndex = (colorIndex + 1) % colors.count
            return color
        }
    }()
    let attributedString = NSAttributedString.attributedStringWithTextsAndColors([nickName, text], colors: [nickNameColor, UIColor(hex: .RGB333333)!])
    return attributedString
}
```


`NSAttributedString.attributedStringWithTextsAndColors`是自己扩展的一个方法，传入多串文字和对应的字符返回匹配的`NSAttributedString`。

主要逻辑是：先判断是否已经有保存过昵称对应的颜色值，有则直接返回；没有则根据`index`获取颜色值，然后保存起来，并改变`index`。



# 点赞动画

iOS自带了粒子引擎的类`CAEmitterLayer`，是一个粒子发射器系统，每个粒子都是`CAEmitterCell`的实例。可以查看它们分别有什么属性。

有两个小点，一个是`CAEmitterLayer`一些属性对`CAEmitterCell`有成倍作用，如`birthRate`；另一个是没有明确的停止动画的方法，包括它的父类也没提供。可以想到的方法，除了把`layer`抹除掉之外，还可以将`CAEmitterLayer`的`birthRate`设置为0，这样每个`CAEmitterCell`的诞生速率都为0，就不会有动画了。

```
class PraiseEmitterView: UIView {

    private var timer: NSTimer?
    private let emitter: CAEmitterLayer! = {
        let emitter = CAEmitterLayer()
        return emitter
    }()
    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        setup()
    }
    private func setup() {
        emitter.frame = bounds
        emitter.birthRate = 0
        emitter.emitterShape = kCAEmitterLayerLine
        emitter.emitterPosition = CGPointMake(0,CGRectGetHeight(bounds))
        emitter.emitterSize = bounds.size
        emitter.emitterCells = [getEmitterCell(UIImage(named: "comment")!.CGImage!), getEmitterCell(UIImage(named: "flower_15")!.CGImage!)]
        self.layer.addSublayer(emitter)
    }
    func timeoutSelector() {
        emitter.birthRate = 0
    }
    func emit() {
        emitter.birthRate = 2
        timer?.invalidate()
        timer = NSTimer.scheduledTimerWithTimeInterval(2, target: self, selector: #selector(timeoutSelector), userInfo: nil, repeats: false)
    }
    private func getEmitterCell(contentImage: CGImage) -> CAEmitterCell {

        let emitterCell = CAEmitterCell()
        emitterCell.contents = contentImage
        emitterCell.lifetime = 2
        emitterCell.birthRate = 2

        emitterCell.yAcceleration = -70.0
        emitterCell.xAcceleration = 0
        
        emitterCell.velocity = 20.0
        emitterCell.velocityRange = 200.0
        
        emitterCell.emissionLongitude = CGFloat(0)
        emitterCell.emissionRange = CGFloat(M_PI_4)
        
        emitterCell.scale = 0.8
        emitterCell.scaleRange = 0.8
        emitterCell.scaleSpeed = -0.15
        
        emitterCell.alphaRange = 0.75
        emitterCell.alphaSpeed = -0.15

        return emitterCell
    }
}
```



-END-
欢迎到我的博客交流：https://zackzheng.info