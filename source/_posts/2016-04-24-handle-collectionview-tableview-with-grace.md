---
title: 戏说CollectionView、TableView如何优雅应对变化的界面展示
date: 2016-04-24 16:00:00
tags: 
     - iOS
categories: iOS
keywords: iOS
description: 介绍了一种基于枚举来处理CollectionView、TableView的 DataSource 和 Delegate 的方案，以及这种方案带来的便利，降低维护成本，能灵活应对变化需求的特点。
---

# 前言

App里很多页面都是用UICollectionView和UITableView做的，比如一些列表类的、详情类的，甚至是商城首页等等，只要是分块的，特别是根据数据源不同而展示不同的页面，都很适合用，但怎么用，使它容易修改、扩展，是个问题。



**产品经理老李说**：

1. 这一块功能有数据的时候就展示，没有就隐藏
2. 这一块挪到那块下面
3. 这一块XXX功能删了，现在不需要了
4. 之前删的XXX功能补回来吧，然后改成放在XXX功能块下面
5. 这块地方的高度根据内容动态设置
6. ......（数不尽的小数点）

**程序猿懵了**：

1. 咦，我之前这个Section放的是什么，看界面的话不准确，有些隐藏之类的情况，界面上这块东西对应的是哪个Cell，也不确定了。
2. Delegate和DataSource，少的时候只实现三四个方法，多的时候实现七八个，这些都和Section、Cell的各项信息有关，这时候做什么变动都需要改动多处地方，if...else...也就写得到处都是。
3. ......（数不尽的小数点）

这些麻烦不难解决，就是有时候会很花时间，效率低，于是我想了一个方法解决它。

最初是在做某一类详情的时候想出来的，后面经过应用到其他页面和几次变更的实践之后发现，还真挺方便的！下面将举几个例子来说明这个方法及其特点。



本想将本文语言组织得有趣些，但可惜功力不够，时间也紧哈，下次再尝试。



# 例子一

*产品经理老李*说：“给我实现xxx功能，有5部分信息，其中A、C、D功能没有东西的时候是隐藏的......”

*程序猿*答道：“哦......”

**转化成技术需求**：

> 有五种Cell，分别是ACell（根据data.a判断隐藏与否）、BCell（默认有）、CCell（根据data.c数组个数判断隐藏与否，并且根据个数设置cell大小）、DCell（根据data.d数组个数判断隐藏与否）、ECell（默认有），隐藏与否顺序都是A\B\C\D\E。

**笨拙的实现**：

```
class XXXController: UICollectionViewController {

    let data = ......
    override func viewDidLoad() {

        super.viewDidLoad()
        collectionView.reloadData()
    }
}

extension XXXController: UICollectionViewDelegateFlowLayout {

    func collectionView(collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAtIndexPath indexPath: NSIndexPath) -> CGSize {

        // switch 或 if...else...
        switch indexPath.row {
        case 0:
            // 判断是A还是B
        case 1:
            // 判断是B还是C还是D还是E......(心里骂着产品经理)
            // 这里很容易犯错，细想下你就知道了......(又被扣工资了囧)
        case 2:
            // 判断是C还是D还是E......(心里骂着产品经理)
        case 3:
            // 判断是D还是E......(心里骂着产品经理)
        case 4:
            // ECell
        default:
            // 还要写default......
        }
    }
}

extension XXXController {// UICollectionViewDelegate、UICollectionViewDataSource

    override func collectionView(collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        // 根据数据源计算分别有没有A/C/D
        // 一堆if...else......(心里骂着产品经理)
        return 2 + ......
    }

    override func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
        // 同sizeForItemAtIndexPath(心里骂着产品经理)
    }

    override func collectionView(collectionView: UICollectionView, didSelectItemAtIndexPath indexPath: NSIndexPath) {
        // 同sizeForItemAtIndexPath(心里骂着产品经理)
    }
}
```

**优雅的实现**：

```
enum XXXItemTypes {
    case A
    case B
    case C(count:Int)
    case D
    case E
    ......
}

class XXXController: UICollectionViewController {

    let data = ......
    var itemTypes:[XXXItemTypes] = []
    override func viewDidLoad() {

        super.viewDidLoad()
        configItemTypes(data)
        collectionView.reloadData()
    }
}

private extension XXXController {
    func configItemTypes(data) {

        itemTypes = []
        if let _ = data.a {// 根据对象是否存在判断隐藏与否的情况
            itemTypes.append(XXXItemTypes.A)
        }
        itemTypes.append(XXXItemTypes.B)// 默认有的情况
        if let cCount = data.c.count where cCount > 0 {// 根据个数判断隐藏与否的情况
            itemTypes.append(XXXItemTypes.C(cCount))
        }
        if let dCount = data.d.count where dCount > 0 {
            itemTypes.append(XXXItemTypes.D)
        }
        itemTypes.append(XXXItemTypes.E)// 默认有的情况
        ......
    }
}

extension XXXController: UICollectionViewDelegateFlowLayout {

    func collectionView(collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAtIndexPath indexPath: NSIndexPath) -> CGSize {

        switch itemTypes[indexPath.row] {
        case .A:
            return ACell.cellSize(......)
        case .B(let count):
            return BCell.cellSize(count: count)
        case .C:
            return CCell.cellSize(......)
        case .D:
            return DCell.cellSize(......)
        case .E:
            return ECell.cellSize(......)
        }
    }
}

extension XXXController {// UICollectionViewDelegate、UICollectionViewDataSource

    override func collectionView(collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return itemTypes.count
    }

    override func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {

        switch itemTypes[indexPath.row] {
        case .A:
            ......
        case .B:
            ......
        case .C:
            ......
        case .D:
            ......
        case .E:
            ......
        }
    }

    override func collectionView(collectionView: UICollectionView, didSelectItemAtIndexPath indexPath: NSIndexPath) {
        switch itemTypes[indexPath.row] {
            ......
        }
    }
}
```

**小分析**：

可以看到，所有的判断（4处地方）都归结一处，也就是configItemTypes里面，省去了一堆因为隐藏与否而做的判断逻辑，而且不会出现复杂的判断的逻辑出错。



# 例子二

又是*产品经理老李*说：“D功能很重要，放在第一处吧，这很容易实现吧！”

*程序猿*答道：“嗯......”

**笨拙的实现**：

每个UICollectionViewDelegate、UICollectionViewDataSource、UICollectionViewDelegateFlowLayout的方法里面的判断逻辑都修改......

（*磨刀霍霍向老李*）

**优雅的实现**：

只需修改configItemTypes，将DCell的判断逻辑放到最前面先判断。（这是真的吗？？就这么简单？？）

**小分析**：

任何顺序变化都可以通过简单修改configItemTypes而实现，不改动其他地方。



# 例子三

*老李*又来了：“把C功能去掉......”

*程序猿*答道：“嗯......”

**笨拙的实现**：

同例子二，又要重新看那段判断逻辑。

**优雅的实现**：

同样只需修改configItemTypes，将添加CCell的部分删除。（好崇拜自己啊！！！！！！！！）

**小分析**：

任何增加和删除都可以通过简单修改configItemTypes而实现，不改动其他地方。



# 进阶例子

上面的几个例子都是没有Section的，如果再加多一层Section，那么复杂的程度就上升很多了，这里就不再赘述了。主要说下这种情况下枚举的数据结构设计，可以有很多种，下面是我的一个实现，还可以更优化。

```
enum XXXSectionType {
    case SectionA(rowTypes:[XXXRowType])
    case SectionB(rowTypes:[XXXRowType], index: Int)
    case SectionC(rowTypes:[XXXRowType])
}

enum XXXRowType {
    case RowA
    case RowB
    case RowC
}
```

这种枚举的方法在有section的情况下更彰显了它的价值！依着上面的几个例子的需求尝试下就能体会得出来了。

# 总结

从上面的几个需求和处理可以慢慢抽象出这麻烦的问题所在：
=>Section/Cell的分布情况没有统一
=>导致Delegate和DataSource的多处方法需要进行判断
=>导致展示变化时Delegate和DataSource的多处方法都要修改
=>导致修改辨识麻烦
=>导致效率低

枚举的方法将所有分布情况都放在了分布数组的初始化上，Delegate/DataSource不需要通过indexPath去判断自己当前是哪个row/item，实际上就是将未知的麻烦的indexPath抽象出一维数组或者二维数组来表示它们的分布，并且每个分布情况所需的设置（如Cell高度、Cell样式、Header高度等等）都是固定的，所以任何变动只需修改那个分布数组。

看似简单的抽取，却处理掉了很多麻烦的问题和冗余的处理！



-END-
欢迎到我的博客交流：https://zackzheng.info