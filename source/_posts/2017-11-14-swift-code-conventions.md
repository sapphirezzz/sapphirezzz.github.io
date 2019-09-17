---
title: 团队Swift编码规范分享
date: 2017-11-14 11:40:00
tags: 
     - swift
     - iOS
categories: iOS
keywords: swift 编码规范 CodeConventions
description: 分享团队在swift项目上的编码规范
---

# 目录

- 命名
- 格式
- 准则
- 文件
- 场景
- 参考

# 命名

**【强制】命名清晰，保持一致性**

反例：displayName（返回name还是展示name）

**【强制】代码中的命名严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式**

说明：正确的英文拼写和语法可以让阅读者易于理解，避免歧义。注意，即使纯拼音命名方式也要避免采用。

正例：alibaba / taobao / youku / hangzhou 等国际通用的名称，可视同英文

反例：DaZhePromotion [打折]、getPingfenByName() [评分] / int 某变量 = 3

**【强制】类、结构体、枚举、协议名使用大驼峰风格，常用缩写除外**

正例：UserManager、UMSocialAdapter、TCPManager

**【强制】函数、方法、变量、常量、参数使用小驼峰命名**

**【强制】协议名统一规范**

作用为delegate，结尾添加Delegate；描述协议做的事，用名词描述；描述行为，用形容词，例如"able"或者"ing"等；如果两者不能满足，结尾添加Protocol。

正例：ScrollToTopable、UIDataSourceTranslating

**【强制】缩略词使用完整大写，如果作为命名的开始部分，且首字母需要小写，则缩略词全小写**

正例：

```swift
let userID = "123456"
let imageURL = "http://xxxxx"
class URLHandler {
  func urlConvert(urlString: String) {}
}
```

**【强制】枚举项小写**

正例：

```swift
enum CompassPoint {
  case north
  case south
  case east
  case west
}
```

**【强制】命名先保证表达的意思准确，再考虑简短，不为了缩短书写而缩短书写**

说明：比如有些属性适合定义成类属性；有些准确的需要定义成实例属性。

正例：UIDevice.current.isIphoneX

反例：UIDevice.isIphoneX

**【强制】常量、变量命名若不能明显表明类型，则属性命名内要包括类型信息**

正例：

```swift
// 变量名中可以表明类型，则命名中不需要包括类型信息
let animationDuration: NSTimeInterval
let userName: String  
// Controller命名 
// UIViewController、UITableViewController等简写成Controller
let popupController: UIViewController 
// UITabBarController和UINavigationController保留结构缩写
let flashSaleTabBarController: UITabBarController
// 命名中需要明确类型信息
let userImage: UIImage
let userImageURL: NSURL
let userImageURLString: String
// 当使用outlets时, 确保命名中标注类型
@IBOutlet weak var submitButton: UIButton!
@IBOutlet weak var emailTextField: UITextField!
@IBOutlet weak var nameLabel: UILabel!
```

**【推荐】如果使用到了设计模式，建议在类名中体现出具体模式**

说明：将设计模式体现在名字中，有利于阅读者快速理解架构设计思想。

正例：class CommunityFollowTopicCellFactory、class TopicListCellBaseFactory

**【强制】如果枚举类型的属性，其命名不能表明是枚举类型的，带上Enum后缀**

正例：UserIdentityEnum、sectionType、playStatus、selectionStyle

**【强制】声明属性时言简意赅，不带上类名**

正例：

```swift
// 系统 UIDevice.current
// 而不是UIDevice.currentDevice
UserDefaults.standard // 而不是UserDefaults.standardDefaults
class RouteManager {
  static let shared = RouteManager() // 用shared，不用sharedManager 
}
```

**【强制】区分使用default或shared**

说明：default一般用于提供使用默认的参数配置的实例；shared一般用于单例。

**【强制】如果省略外部参数名后会导致调用处含义模糊，则禁止省略**

反例：

```swift
// 方法声明
class func action(_ pageName: String?, pageParam: String?, action: String, actionParam: [String: Any]?, isOutPoint: Bool = false) {} 
// 调用处 UserTracker.action("搜索", pageParam: searchType.rawValue, action: "语音输入", actionParam: ["keywords": result], isOutPoint: true)
```

**【强制】函数命名中尽量不添加介词，如of、in、on、with等**

正例：

```swift
// 原声明：UIFont.systemFontOfSize
open class func systemFont(ofSize fontSize: CGFloat) -> UIFont // 原声明：UIView.animateWithDuration
open class func animate(withDuration duration: TimeInterval, animations: @escaping () -> Swift.Void)
```

# 格式

**【强制】如果语句的逻辑或长度较复杂，则使用变量保存再引用**

反例：

```swift
AnyDataSourceService(dataSource: GoodsCommentResultDataSource(id: goodsId, offset: offset, imageOnly: imageOnly)).run(success: {
  [weak self] (data: GoodsCommentResult) in
})
```

正例：

```swift
let dataSource = GoodsCommentResultDataSource(id: goodsId, offset: offset, imageOnly: imageOnly)
let service = AnyDataSourceService(dataSource: dataSource)
service.run(success: { [weak self] (data: GoodsCommentResult) in
})
```



**【强制】如果大括号内为空，则简洁地写成{}即可，不需要换行**

**【强制】枚举每一个case操作都换行，不跟在“：”后面**

```swift
switch enum {
case a:
  methodA()
case b:
  methodB()
case c:
  methodC()
}

```

**【推荐】如果switch内每一个case的操作大于5行，则封装成一个方法调用**

**【强制】变量类型，函数参数，遵循协议或继承父类，分号前不留空格**

正例：

```swift
let str: String = "Test"
someFunction(someArgument: "Argument")
class ViewController: UIViewController {}
extension ViewController: UITableViewDelegate {}

```

**【强制】逗号后面、运算符前后加空格**

正例：

```swift
let array = [1, 2, 3, 4, 5]
let sum = 1 + 2
let isSuccess = sum == 3

```

**【强制】左大括号不换行，左边保留空格**

**【强制】流程控制不使用小括号**

正例：if x == y { }

**【强制】使用枚举时用简写**

正例：imageView.setImageWithURL(url, type: .person)

**【强制】使用一些语句如 else，catch等紧随代码块的关键词的时候，确保代码块和关键词在同一行**

正例：

```swift
do {
  try canThrowAnError()     // no error was thrown 
} catch {
  // an error was thrown 
} 
if name == "world" {
  print("hello, world")
} else {
  print("I'm sorry \(name), but I don't recognize you")
}

```

**【强制】switch与case对齐**

正例：

```swift
switch some value to consider {
case value 1:
  respond to value 1
case value 2, value 3:
  respond to value 2 or 3
default:
  otherwise, do something else 
}

```

**【强制】不注释无用代码，直接删掉。若想保留代码以防以后用到，请使用git**

**【强制】文件末尾必须留且只留一行空白行**

**【强制】“//”注释符号后面要保留空格**

**【强制】求高度/字符串等较复杂时需按一下格式定义，清晰指明含义，方便他人维护**

```swift
let factor1Top = 20
var factor1Height = 40
var factor2Height = 40
let bottomPadding = 30
if lineCount > 0 {
  let lineHeight = lineCount * 10
  factor1Height += lineHeight
}
if lineCount > 2 {
  let lineHeight = lineCount * 20
  factor2Height += lineHeight
}
let height = factor1Top + factor1Height + factor2Height + bottomPadding
return CGSize(width: width, height: height)

```

# 准则

**【强制】若变量类型可以依靠推断得出，则声明时不要指明类型**

正例：

```swift
let π = 3.14159

```

**【强制】模型中需要指明数据类型**

正例：

```swift
struct DiamondPackage {
  var id: Int = 0
  var count: Int = 0
  var price: Double = 0.0
  var desc: String?
  var descIconURL: NSURL?
  var iapProductId: String?
}

```

**【强制】使用隐式拆包可选类型的场景只能是@IBOutlets和网络层Service（保证使用之前肯定有值非空时），其余情况禁止使用“!”隐式拆包**

**【强制】若需要判断当前值是否为nil，直接和nil比较**

正例：if someOptional != nil {}

反例：if let _ = someOptional {}

**【强制】使用属性时不用self.修饰**

**【强制】使用guard代替if提前返回**

**【强制】使用guard拆包多个可选值**

正例：

```swift
guard let thingOne = thingOne, let thingTwo = thingTwo, let thingThree = thingThree else {     
  return 
}

```

**【强制】严格设置访问权限open／public／internal／fileprivate／private**

**【强制】不使用的库不import进文件**

说明：一般新建文件之后都会有默认代码：import Foundation，不需要则删除

# 文件

**【强制】Controller文件结构**

说明：Controller包含较多代码，需要适量划分，使得代码查找更方便

一般包含以下内容：

```swift
import
@IBOutlet
@IBAction
override
配置数据源（configSection和enum Row）
私有属性 开放属性 私有函数 开放函数
delegate/protocol

```

正例：

```swift
// import放最前面，先import系统库
import UIKit
import Alamofire
protocol ViewControllerDelegate: class {}
class ViewController: UIViewController {
  // 仅包含@IBOutlet、@IBAction、私有属性、公有属性、override方法
  // 顺序依次如下     
  // IBOutlet
  @IBOutlet var imageView: UIImageView!
  // 公有属性
  var showBottom: Bool = false       
  // 私有属性     
  private weak var delegate: ViewControllerDelegate?
  private var rows: [Row] = []
  override func viewDidLoad() {
    super.viewDidLoad()
    configVM()
    doSomething()
    // Do any additional setup after loading the view, typically from a nib.
  }
  // override按生命周期顺序
  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
  }
  override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
  }
}  
// 每个extension内第一行空行
// @IBAction 
extension ViewController {

  @IBAction func clickButton() {}
}
// 类方法、开放的方法
extension ViewController {
  class func classMethod() {}
  func openMethod() {}
}
// 私有的方法 
private extension ViewController {
  func doSomething() {}
}
// 协议，每个协议分开写

extension ViewController: UITableViewDelegate {} 
extension ViewController: ScrollToTop {}
// 数据源配置相关放最后的extension
private extension ViewController {
  enum Row {
    case row1
    case row2
    case row3
  }
  func configVM() {
    rows = []
    rows.append(.row1)
  }
}

```

# 场景

- **定义模型**

【强制】唯一标识统一用id，不使用类似jobId写法

【强制】不参与运算的String不需要初始化，如name,desc等仅用于显示的字段

【强制】数组必须初始化，不使用optional写法

- **使用CocoaPods**

【强制】导入库需要指定某一个确定的版本号，禁止使用大于等于之类的指定

【强制】修改第三方库（禁止修改，除非特殊情况）需要新建一个Pod，并且在提交podfile修改的commit中注释原因

【强制】不轻易引入第三方库，除了网络库、JSON转模型库、路由库等

【强制】保持相同作用的库仅有一个

【推荐】团队开发的库尽量引入framework，不引入源码

# 参考

[swift-style-guide](https://github.com/linkedin/swift-style-guide#1-code-formatting)

[最详尽的 Swift 代码规范指南](http://www.cocoachina.com/swift/20160725/17176.html)

[17条 Swift开发规范 最佳实践](http://mobile.51cto.com/news-493482.htm)

[The Swift Programming Language (Swift 4)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID309)



-END-
欢迎到我的博客交流：https://zackzheng.info