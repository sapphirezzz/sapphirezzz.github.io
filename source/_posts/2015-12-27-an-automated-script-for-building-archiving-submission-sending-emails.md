---
title: 详解Shell脚本实现iOS自动化编译打包提交
date: 2015-12-27 16:00:00
tags: 
     - iOS
     - 持续集成
     - shell
categories: iOS
keywords: iOS Shell 持续集成
description: 详解Shell脚本实现iOS自动化编译打包提交的过程
---

# 目录

- 前言
- Shell脚本涉及的工具
  - xcodebuild和xcrun
  - altool
  - fir-cli
  - PlistBuddy
- 一些概念的区别
- 具体实现
  - xcodebuild和xcrun
  - 准备Plist文件
  - 获取命令行参数
  - 清理构建目录
  - 编译打包成Archive
  - 将Archive导出
  - 上传到Fir
  - 验证并上传到App Store
  - 邮件通知相关同事
  - 上传符号表到Bugly
- 简单例子
- 对比实验
  - 三种方式的对比
  - xcodebuild+xcrun和仅xcodebuild的比较
  - 命令到底做了什么
- 总结

# 前言

现在涉及到编译打包的工作主要是以下两个：
1. 提交测试版本给测试同事
2. 提交App Store审核

两个流程分别是：

- 修改证书和配置文件，然后「Product -> Archive」编译打包，之后在自动弹出的 「Organizer」 中进行选择，根据需要导出 ad hoc enterprise 类型的 ipa 包。等待导出之后再提交到Fir上，等Fir提交完成就需要告知测试同事。整个流程下来一般都要半个多小时，而且需要人工监守操作。
- 第二个也是差不多，打包完之后需要操作几个步骤然后上传到App Store，上传时间较长，而且中间可能会有错误需要处理。上传后等待苹果处理二进制包，苹果处理后上去选择构建包，点击提交审核。

所以研究下自动化编译打包，提高下效率，减少人工操作成本。

主要有两种实现途径，AppleScript和Shell脚本，`AppleScript`没怎么研究，网上说是很强大的脚本语言。

下面主要讲Shell脚本的实现，网上也有人实现了并托管在`github`上，可以参考下。
> https://github.com/webfrogs/xcode_shell

# Shell脚本涉及的工具

主要是以下几个工具：

1. xcodebuild
2. xcrun
3. altool（提交到App Store使用）
4. fir-cli（上传到fir时使用）
5. Python的smtplib（之前已经写过python的发邮件了，所以就直接用没有用Shell写。）
6. PlistBuddy
7. BuglySymboliOS（Bugly的符号表工具包）

## xcodebuild和xcrun
`xcodebuild`和`xcrun`都是来自`Command Line Tools`，Xcode自带，如果没有可以通过以下命令安装：

``` 
xcode-select --install
```

或者在下面的链接下载安装：
>  https://developer.apple.com/downloads/ 

安装完可在以下路径看到这两个工具：
> /Applications/Xcode.app/Contents/Developer/usr/bin/

- xcodebuild
主要是用来编译，打包成Archive和导出ipa包。

> https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html

可以执行 `xcodebuild -help` 查看，主要展示了几种用法、一些可选项，最后是比较重要的exportOptionsPlist文件的一些可选key，这个文件在后面导出ipa包会用到。

主要下面三个查看的命令比较重要：
``` 
-showsdks                           display a compact list of the installed SDKs
-showBuildSettings                  display a list of build settings and values
-list                               lists the targets and configurations in a project, or the schemes in a workspace
```

后面两个需要在Xcode的project或者workspace目录下才能用。

- xcrun
``` 
xcrun -h
```

主要是打包，看网上比较多是用这个工具打包各种渠道包。

## altool

这个工具在网上搜索几乎没有什么结果，大概国内直接用命令行工具提交App Store的比较少。后来在StackOverflow上才找到相关的文档：
> https://itunesconnect.apple.com/docs/UsingApplicationLoader.pdf

在上面的文档第38页讲述了如何使用altool上传二进制文件。

这个工具实际上是ApplicationLoader，打开Xcode-左上角Xcode-Open Developer Tool-Application Loader 可看到。有个“交付您的应用”操作，网上看到有人是直接用这个工具上传的。

altool的路径是：

> /Applications/Xcode.app/Contents/Applications/Application\ Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Support/altool

使用时会提示下面的错误：
``` 
altool[] *** Error: Exception while launching iTunesTransporter: 
Transporter not found at path: /usr/local/itms/bin/iTMSTransporter. 
You should reinstall the application.
```

建立个软链接可解决（类似于Windows的快捷方式）：
``` 
ln -s /Applications/Xcode.app/Contents/Applications/Application\ Loader.app/Contents/itms /usr/local/itms
```

## fir-cli
安装时会提示各种权限不允许，可以执行下面命令：

``` 
echo 'gem: --bindir /usr/local/bin' >> ~/.gemrc
sudo 'gem install fir-cli
```

fir有提供Android Studio、Eclipse、gradle插件，可以看下。
> http://fir.im/tools

这是它的github地址，其中讲到有对?`xcodebuild`?原生指令进行了封装。
> https://github.com/FIRHQ/fir-cli/blob/master/README.md

## PlistBuddy

Plist在Mac OSX系统中起着举足轻重的作用，系统和程序使用Plist文件来存储自己的安装/配置/属性等信息。而PlistBuddy是Mac里一个用于命令行下读写plist文件的工具，在/usr/libexec/下。可以通过它读取或修改plist文件的内容。

这里我仅通过它来获取内部版本号、外部版本号。在一些文章中见过用来修改plist文件的信息来导出出不同需要的包。

# 一些概念的区别

Workspace、Project、Scheme、Target的区别。

下面是官方文档：
> https://developer.apple.com/library/ios/featuredarticles/XcodeConcepts/Concept-Targets.html#//apple_ref/doc/uid/TP40009328-CH4-SW1

下面从上往下大概说下，具体看文档比较好：

- Workspace
`Workspace`是最大的集合，可以包含多个`Project`，可以管理不同的`Project`之间的关系。`Workspace`是以`xcworkspace`的文件形式存在的。（这点和`Project`一致）。`Workspace`的存在是为了解决原来仅有`Project`的时候不同的`Project`之间的引用和调用困难的问题。同时，一个`Workspace`的`Project`共用一个编译路径。比如使用CocoaPod、或者使用其他开发库/框架。

- Project
`Project`是一个仓库，包含编译一个或多个`product`所需的文件、资源和信息，保持和聚合这些元素间的关系。（每个`Target`能指定自己的`Build Settings`来覆盖`Project`的）

> - Source code, including header files and implementation files
> - Libraries and frameworks, internal and external
> - Resource files
> - Image files
> - Interface Builder (nib) files

- Scheme
`Scheme`包含了一些要构建的Scheme，一些构建时用到的设置，一些要运行的测试。同时只能有一个`Scheme`是有效的。

- Target
`Target`是对应了具体一个想要构建的`Product`,包含了一些构建这个`Product`所需的配置和文件（`build settings`和`build phases`）。一个`Project`可以包含多个`Target`。

# 具体实现

看起来有两种实现方法：
- 网上可以查到的文章，大多数都是用`xcodebuild`和`xcrun`实现的，比如：
``` 
xcodebuild -workspace XXX -scheme XXX -configuration Release
xcrun -sdk iphoneos PackageApplication -v "/XXX/XXX.app" -o "/XXX/XXX"
```

这些文章都是相对比早期的，大多数用于打包不同渠道包。

- 另一种是`xcodebuild`的`archive`和`-exportArchive`，只有一两篇文章是用这个，而且也过时了，因为现在最新是需要用`-exportOptionsPlist`这个选项。

我用的是第二种，并用上`-exportOptionsPlist`选项，后面我会简单给下这两种的结果比较。脚本流程是：

1. 准备两个`Plist`文件，用于导出不同`ipa`包时使用。
2. 获取命令行参数，区分上传到`Fir`还是`App Store`
3. 清理构建目录
4. 编译打包
5. 导出包
6. 上传到`Fir`或者验证并上传到`App Store`
7. 发邮件通知

## 准备Plist文件

根据`xcodebuild -help`提供的可选key可以知道，`compileBitcode`、`embedOnDemandResourcesAssetPacksInBundle`、`iCloudContainerEnvironment`、`manifest`、`onDemandResourcesAssetPacksBaseURL`、`thinning`这几个key用于非`App Store`导出的；`uploadBitcode`、`uploadSymbols`用于`App Store`导出；`method`、`teamID`共用。

method的可选值为:
> app-store, package, ad-hoc, enterprise, development, and developer-id

所以我建了两个文件：`AppStoreExportOptions.plist`、`AdHocExportOptions.plist`。

AppStoreExportOptions.plist：method＝app-store，uploadBitcode＝YES，uploadSymbols＝YES

AdHocExportOptions.plist：method＝ad-hoc，compileBitcode＝NO

## 获取命令行参数
用`Shell`内置的`getopts`命令，这属于Shell的范畴就不多讲了：

``` 
if [ $# -lt 1 ];then
    echo "Error! Should enter the archive type (AdHoc or AppStore)."
    echo ""
    exit 2
fi
while getopts 't:' optname
do
    case "$optname" in
    t)
        if [ ${OPTARG} != "AdHoc" ] && [ ${OPTARG} != "AppStore" ];then
            echo "invalid parameter of $OPTARG"
            echo ""
            exit 1
        fi
        type=${OPTARG}
        ;;
    *)
        echo "Error! Unknown error while processing options"
        echo ""
        exit 2
        ;;
    esac
done
```

## 清理构建目录
就如在Xcode操作「Product -> Clean」。

``` 
log_path="/XXX/XXX"
configuration="Release"
xcodebuild clean -configuration "$configuration" -alltargets >> $log_path
```

log_path是一个文档路径，只是用来记录命令的输出，因为都打在终端会很多，另外也方便后面分析。后面的命令也是如此。这里面带的选项可以根据需要参考`xcodebuild -help`的信息。

## 编译打包成Archive

就如在Xcode操作「Product -> Archive」

``` 
workspaceName="XXX.xcworkspace"
scheme="XXX"
configurationBuildDir="XXX/build"
codeSignIdentity="iPhone Distribution: XXX, Ltd. (xxxxxxxxxx)"
adHocProvisioningProfile="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
appStoreProvisioningProfile="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
configuration="Release"
archivePath="/xxx/XXX.xcarchive"

xcodebuild archive -workspace "$workspaceName" -scheme "$scheme" -configuration "$configuration" -archivePath "$archivePath" CONFIGURATION_BUILD_DIR="$configurationBuildDir" CODE_SIGN_IDENTITY="$codeSignIdentity" PROVISIONING_PROFILE="$provisioningProfile" >> $log_path
```

这里的`CONFIGURATION_BUILD_DIR`是中间文件生成的路径，可以不指定；`CODE_SIGN_IDENTITY`是证书名（在对应`TARGETS`的`Build Settings`中选择完`Code Sinning`，再点击选择`Other...`，就可以得到这串东西）；`PROVISIONING_PROFILE`是配置文件（获取方法同CODE_SIGN_IDENTITY，格式一般是`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`）。还可以添加其他参数，不设置的都是默认使用项目Build Settings里面的配置，包括`CODE_SIGN_IDENTITY`和`PROVISIONING_PROFILE`。

如果是workspace就用`-workspace`，就像编译带有`CocoaPods`的项目，如果是普通项目则用`-project`。

执行完会生成一个.xcarchive文件和build文件夹如下：

``` 
.xcarchive
build文件夹
	|------.a
	|------.app
	|------.app.dSYM
	|------.swiftmodule文件夹
		|------arm.swiftdoc
		|------arm.swiftmodule
		|------arm64.swiftdoc
		|------arm64.swiftmodule
```

## 将Archive导出

``` 
xcodebuild -exportArchive -archivePath "$archivePath" -exportOptionsPlist "$exportOptionsPlist" -exportPath "/XXX/XXX" >> $log_path
```

其中`$exportOptionsPlist`是对应使用的Plist的完整路径（包括文件名）。

然后就会在指定的`exportPath`路径下生成.ipa文件。

## 上传到Fir

``` 
firApiToken="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
ipaPath="/xxx/xxx.ipa"
fir publish "$ipaPath" -T "$firApiToken" >> $log_path
```

`firApiToken`在登录Fir后，右上角-API token看到。

## 验证并上传到App Store

``` 
altoolPath="/Applications/Xcode.app/Contents/Applications/Application\ Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Versions/A/Support/altool"
${altoolPath} --validate-app -f ${ipaPath} -u xxxxxx -p xxxxxx -t ios --output-format xml >>
${altoolPath} --upload-app -f ${ipaPath} -u xxxxxx -p xxxxxx -t ios --output-format xml
```

在上面的PDF文档第38页讲明了用法和各个可选项，具体可以看下PDF。需要说明的是，生成的结果是xml打印在终端，可以保存到文档再解析出key来判断是否成功，目前这步还没做。

这是成功的结果：

``` 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>os-version</key>
	<string>10.11.2</string>
	<key>success-message</key>
	<string>No errors validating archive at /XXX/XXX.ipa</string>
	<key>tool-version</key>
	<string>1.1.902</string>
	<key>xcode-versions</key>
	<array>
		<dict>
			<key>path</key>
			<string>/Applications/Xcode.app</string>
			<key>version.plist</key>
			<dict>
				<key>BuildVersion</key>
				<string>7</string>
				<key>CFBundleShortVersionString</key>
				<string>7.2</string>
				<key>CFBundleVersion</key>
				<string>9548</string>
				<key>ProductBuildVersion</key>
				<string>7C68</string>
				<key>ProjectName</key>
				<string>IDEFrameworks</string>
				<key>SourceVersion</key>
				<string>9548000000000000</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
```

这是失败的结果（找不到iTMSTransporter的情况，用前面说的ln -s解决）：

``` 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>os-version</key>
	<string>10.11.2</string>
	<key>product-errors</key>
	<array>
		<dict>
			<key>code</key>
			<integer>-10001</integer>
			<key>message</key>
			<string>Transporter not found at path: /usr/local/itms/bin/iTMSTransporter.  You should reinstall the application.</string>
			<key>userInfo</key>
			<dict>
				<key>MZUnderlyingException</key>
				<string>Transporter not found at path: /usr/local/itms/bin/iTMSTransporter.  You should reinstall the application.</string>
				<key>NSLocalizedDescription</key>
				<string>Transporter not found at path: /usr/local/itms/bin/iTMSTransporter.  You should reinstall the application.</string>
				<key>NSLocalizedFailureReason</key>
				<string>Transporter not found at path: /usr/local/itms/bin/iTMSTransporter.  You should reinstall the application.</string>
			</dict>
		</dict>
	</array>
	<key>tool-version</key>
	<string>1.1.902</string>
	<key>xcode-versions</key>
	<array>
		<dict>
			<key>path</key>
			<string>/Applications/Xcode.app</string>
			<key>version.plist</key>
			<dict>
				<key>BuildVersion</key>
				<string>7</string>
				<key>CFBundleShortVersionString</key>
				<string>7.2</string>
				<key>CFBundleVersion</key>
				<string>9548</string>
				<key>ProductBuildVersion</key>
				<string>7C68</string>
				<key>ProjectName</key>
				<string>IDEFrameworks</string>
				<key>SourceVersion</key>
				<string>9548000000000000</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
```

可见，成功会有个`success-message`的key，而失败会有`product-errors`的key。

## 邮件通知相关同事

发邮件时可能会想带上当前版本的一些信息，如版本号、内部版本号等，可以用PlistBuddy实现读取甚至修改Plist文件。

``` 
appInfoPlistPath="`pwd`/xxx/xxx-Info.plist"
bundleShortVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleShortVersionString" ${appInfoPlistPath})
bundleVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" ${appInfoPlistPath})
```

之后便是发邮件：

``` 
python sendEmail.py "测试版本 iOS ${bundleShortVersion}(${bundleVersion})上传成功" "赶紧下载体验吧！http://fir.im/meijia"
```

或者
``` 
python sendEmail.py "正式版本 iOS ${bundleShortVersion}(${bundleVersion})提交成功" "iOS ${bundleShortVersion} 提交成功！"
```

python主要用smtplib，网上的文章大多都是旧的，特别是讲到SSL时特别复杂，其实具体看下smtplib的接口文档就可以实现了。另外有可能出现标题、内容乱码的现象。整合了下面的链接解决了：

[python 发送邮件解决所有乱码问题]: http://outofmemory.cn/code-snippet/1464/python-send-youjian-resolve-suoyou-luanma-question
[Unicode和Python的中文处理]: http://www.cnblogs.com/TsengYuen/archive/2012/05/22/2513290.html

下面是实现了SSL Smtp登录的。

``` 
#!/usr/bin/env python3
#coding: utf-8

# sendEmail title content
import sys
import smtplib
from email.mime.text import MIMEText
from email.header import Header

sender = 'xxxxxx@qq.com;'
receiver = 'xxx@qq.com;'
smtpserver = 'smtp.qq.com'
#smtpserver = 'smtp.exmail.qq.com'

username = sender
password = 'xxxxxx'

def send_mail(title, content):

    try:
        msg = MIMEText(content,'plain','utf-8')
        if not isinstance(title,unicode):
            title = unicode(title, 'utf-8')
        msg['Subject'] = title
        msg['From'] = sender
        msg['To'] = receiver
        msg["Accept-Language"]="zh-CN"
        msg["Accept-Charset"]="ISO-8859-1,utf-8"

        smtp = smtplib.SMTP_SSL(smtpserver,465)
        smtp.login(username, password)
        smtp.sendmail(sender, receiver, msg.as_string())
        smtp.quit()
        return True
    except Exception, e:
        print str(e)
        return False

if send_mail(sys.argv[1], sys.argv[2]):
    print "done!"
else:
    print "failed!"
```

可以赋值给msg['CC']实现抄送，经过测试，抄送的人过多会有一部分不成功，网上查了是这个库的bug。发送多个人用分号，另外末尾也要用分号。

## 上传符号表到Bugly

用于分析解决崩溃bug挺好用的，而且他们的客服也很及时。
发现他们的2.4.1版本有问题，反馈后他们给了2.4.3版本，经测试没问题。

1. 在Bugly官网下载[符号表工具](https://bugly.qq.com/whitebook)

2. 设置settings.txt

3. 调用命令

```
java -jar buglySymboliOS.jar -d -i $dSYM -u -id "xxxxxxxxx" -key "xxxxxxxxxxx" -package "com.xxx.xxx" -version "$version" ­-o "xxx.zip"
```
注意版本号之类的要设置对。

# 简单例子

清理构建目录：
```
xcodebuild clean -configuration Release -alltargets
```
归档（其他参数不指定的话，默认用的是.xcworkspace或.xcodeproj文件里的配置）
```
xcodebuild archive -workspace xxx.xcworkspace -scheme xxx -configuration Release -archivePath ./xxx.xcarchive
```
导出IPA
```
xcodebuild -exportArchive -archivePath ./xxx.xcarchive -exportOptionsPlist ./AdHocExportOptions.plist -exportPath ./
```
上传FIR
```
fir publish ./xxx.ipa -T xxxxxx
```
提交AppStore
```
/Applications/Xcode.app/Contents/Applications/Application Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Versions/A/Support/altool --validate-app -f ./xxx.ipa -u xxx -p xxx -t ios --output-format xml
/Applications/Xcode.app/Contents/Applications/Application Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Versions/A/Support/altool --upload-app -f ./xxx.ipa -u xxx -p xxx -t ios --output-format xml
```
发邮件
```
python sendEmail.py "邮件内容" "用户名" "密码"
```
上传符号表
```
java -jar buglySymboliOS.jar -d -i $dSYM -u -id "xxxxxxxxx" -key "xxxxxxxxxxx" -package "com.xxx.xxx" -version "$version" ­-o "xxx.zip"
```

# 对比实验

为了了解一些区别，我做了几个对比。我这里定义下三种方式，方便下面说明。

- xcodebuild+xcrun（xcodebuild build和xcrun）
- 只用xcodebuild（archive和exportArchive），
- Xcode。

## 三种方式的对比

我使用xcodebuild+xcrun、仅xcodebuild、Xcode三种分别对相同代码和配置进行操作，根据结果做比较：

- xcodebuild+xcrun

ipa：40.7MB，.app：93.3MB，编译耗时：8m31s，打包耗时：15s。

- 仅xcodebuild

ipa：37.3MB，.app：74MB，.xcarchive：227.3MB，编译耗时：8m24s，打包耗时：26s。

- Xcode

ipa：37.3MB，.app：74MB，.xcarchive：227.3MB，编译耗时：8m40s，打包耗时：30s。

Xcode生成的`.xcarchive`文件可以在以下路径看到：

> /Users/double/Library/Developer/Xcode/Archives

可以看出，<u>仅使用xcodebuild的结果和使用Xcode编译打包的结果是一致的</u>，并且最终的ipa也可以正常安装使用。而第一种xcodebuild+xcrun的结果略大些，但是ipa也是可以正常使用的。这时需要了解下他们的区别。

## xcodebuild+xcrun和仅xcodebuild的比较

- 使用xcrun打包方式二产生的.xcarchive中的.app

打包生成的.ipa文件大小同样为37.3MB，与方式二使用Xcodebuild -exportArchive的结果一致！这样说明：**使用xcrun的打包方法是正常的**，和xcodebuild -exportArchive的结果一致，而且**.ipa包仅和.app有关**。那么说明，<u>这两种方式的不同仅在于xcodebuild build和xcodebuild archive之间的不同</u>。

- 删除.xcarchive中其他文件然后`exportArchive`

这时命令提示错误，但是上面我们已经得出结论.ipa的生成只和.app有关，所以可能的原因是，这个`exportArchive`命令会检查.archive的完整性和正确性，防止生成的.archive不完整或者是伪造的。下面做个实验看下。

## 命令到底做了什么
根据命令运行时输出的内容，看下中间做了什么

- xcrun -sdk iphoneos PackageApplication -v xxx.app -o xxx.ipa

``` 
Packaging application: '/xxx/xxx.app'
Arguments: output=/xxx/xxx.ipa  verbose=1  
Environment variables:
SDKROOT = /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS9.2.sdk
......
SHELL = /bin/bash

Output directory: '/xxx/xxx.ipa'
Temporary Directory: '/var/folders/21/6s9bb23j0s1343pm7ltnlgpm0000gn/T/taOIiK9AyK'  (will NOT be deleted on exit when verbose set)
+ /bin/cp -Rp /xxx/xxx.app /var/folders/21/6s9bb23j0s1343pm7ltnlgpm0000gn/T/taOIiK9AyK/Payload
Program /bin/cp returned 0 : []
### Checking original app
+ /usr/bin/codesign --verify -vvvv /xxx/xxx.app
Program /usr/bin/codesign returned 0 : [/xxx/xxx.app: valid on disk
/xxx/xxx.xcarchive/Products/Applications/xxx.app: satisfies its Designated Requirement
]
Done checking the original app
+ /usr/bin/zip --symlinks --verbose --recurse-paths /Users/double/Desktop/1.ipa .
Program /usr/bin/zip returned 0 : [  adding: Payload/	(in=0) (out=0) (stored 0%)
  adding: Payload/xxx.app/	(in=0) (out=0) (stored 0%)
  ......
```

主要检查了环境变量，然后验证签名，然后压缩（看到了吗，居然是/usr/bin/zip），后面adding的基本都是.nib和.png等的压缩。看起来.archive只是一种压缩形式，包含了.app、.dSYM、.plist和其他一些文件。

这里的`codesign`工具就是签名相关的，可以查看说明：

``` 
SYNOPSIS
     codesign -s identity [-i identifier] [-r requirements] [-fv] [path ...]
     codesign -v [-R requirement] [-v] [path|pid ...]
     codesign -d [-v] [path|pid ...]
     codesign -h [-v] [pid ...]
```

-s是签名，-v是验证。所以可以在.app生成后再签名。

- xcodebuild clean

清理工作，根据参数删除指定的workplace、target、configuration（release或debug） 的中间文件，都是工程目录下的build文件夹。

- xcodebuild archive

下面是里面主要的步骤：

1. Create product structure 创建.app文件
2. CompileC 编译文件（clang编译，指定了编译的SDK版本和指令集）
3. Ld
4. CreateUniversalBinary (lipo)
5. CompileStoryboard (ibtool )
6. CompileAssetCatalog (actool )
7. ProcessInfoPlistFile (builtin-infoPlistUtility )
8. GenerateDSYMFile (dsymutil )
9. LinkStoryboards(ibtool )
10. Strip 
11. ProcessProductPackaging (builtin-productPackagingUtility )
12. CodeSign (codesign --force --sign)
13. Validate (builtin-validationUtility )

# 总结

呼呼写了这么多，终于到总结部分了。这个过程学到了很多东西，脚本成果确实方便了很多，减少了编译打包过程中人工监守、人工操作的成本，并且测试和提交到appStore的包都验证过可用。



-END-
欢迎到我的博客交流：https://zackzheng.info