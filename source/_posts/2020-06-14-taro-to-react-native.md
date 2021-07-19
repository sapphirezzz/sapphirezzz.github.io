---
title: Taro转React-Native的实践总结和分析思考
date: 2020-06-14 22:00:00
tags: 
     - Taro
     - React-Native
categories: Tech
description: 通过实践分享Taro转React-Native遇到的问题，以及相关的思考

---

# 一、前言

> **Taro** 是一套遵循 [React](https://reactjs.org/) 语法规范的 **多端开发** 解决方案。
>
> 使用 **Taro**，我们可以只书写一套代码，再通过 **Taro** 的编译工具，将源代码分别编译出可以在不同端（微信/百度/支付宝/字节跳动/QQ/京东小程序、快应用、H5、React-Native 等）运行的代码。
>
> *摘自[Taro 介绍](https://taro-docs.jd.com/taro/docs/README)

`Taro`是一套优秀的多端开发解决方案。我们团队自较早期时(18年底)便开始了深入研究，并投入到丰富的微信小程序开发实践中，同时作者还负责团队`Taro`转百度/字节跳动/`React-Native`的项目。

本文将介绍`Taro`转`React-Native`的项目框架、`Taro`转`React-Native`过程中遇到的问题和解决方案、RN的热更新，以及框架相关的思考。

项目完成于去年(2019年)，最近才抽空重新修改了总结文档放到博客上。因此可能有些转换细节不是最新的，请留意。

# 二、项目实践

Taro 转 React-Native（以下简称RN） 项目，目前应用于将“美梨工坊桌面”项目（Taro）转换成美梨桌面 iOS App（基于 RN 进行封装的 App 壳），是 App 的 RN 内置包系统，主要功能为 RN 所有功能的展现，包括协作项目开发、页面展示、公共 Bundle、热更新、预下载、一键部署等特性。

作用：让客户端/前端团队能支持更灵活的开发协作、节省开发成本、支持及时处理线上问题。

## 2.1 整体框架

包含以下四个方面。

### 开发链路

- 基于 NPM 的 Web 开发体验
- 基于 Webpack 构建，完整的项目管理
- React-Native 二次组件、工具集合类

### 打包链路

- 自动打包、随版本生成配置
- 持续集成、部署、交付

### 发布链路

- git webhook，自动发布
- 后台 API，OSS 自动上传
- 发布配置后台，通知提醒

### Native 支持

- 推送功能支持
- 原生路由至 RN 页面支持
- Bundle 文件和资源文件的更新

## 2.2 RN 准备工作

目前只在 iOS 平台上做部署，所以只展开 iOS 端的准备工作。

- 安装依赖库

```shell
brew install node
brew install watchman
```

- 安装 React-Native 命令行工具

```shell
npm install -g yarn react-native-cli
```

- 安装 Xcode
- 在 config/index.js 文件中配置：

```json
rn: {
  appJson: {
    name: 'MeiliDesktop',
  },
}
```

相关开发链接：

- [RN开发文档](https://reactnative.cn/docs/getting-started/)
- [Taro React-Native 端开发流程](https://taro-docs.jd.com/taro/docs/react-native.html)

## 2.3 遇到的问题

主要列举 Taro 转换 React-Native 过程中遇到的问题及处理方法。

### 2.3.1 环境/依赖库相关问题

- node-sass

执行项目 yarn，安装 node-sass 库会失败，根据 [node-sass 官网](https://www.npmjs.com/package/node-sass/v/4.9.1)文档，切换源并使用 npm 可以成功：

```
npm install -g mirror-config-china --registry=http://registry.npm.taobao.org
npm install node-sass
```

- fbjs

提示找不到 fbjs，添加该库并执行安装。

- NervJS/taro-native-shell 壳子应用编译失败

文档建议使用 [NervJS/taro-native-shell](https://github.com/NervJS/taro-native-shell) 工程，该工程是 RN 工程中原生的部分。

可能会遇到环境或 Xcode 版本兼容问题，可以自行生成壳子工程。

```
react-native init AwesomeProject
```

- The UMNativeModulesProxy native module is not exported through NativeModules

使用自行生成的壳子工程需要自行安装依赖`react-native-unimodules`，添加到`package.json`的`dependencies`中。

- selector hasGrantedPermission

`react-native-unimodules`使用的是 0.7.0 版本，发现换成 0.6.0 即可解决。

```
"react-native-unimodules": "0.6.0"
```

### 2.3.2 组件/样式问题

- Invariant Violation: Text strings must be rendered within a  component.

文字需要包在 `Text` 组件里面

```
<View className='page-order-detail__content'>
    {steps && steps.length && (<View></View>)
</View>
```

上面的代码也会提示错误，需要改成：

```
<View className='page-order-detail__content'>
    {steps && steps.length > 0 && (<View></View>)
</View>
```

- Input 设置`line-height`会导致输入内容显示异常

内容会显示偏下并被遮挡

- Text 组件不支持设置圆角等
- Input 不支持设置`Height`
- 400,700，normal 或 bold 之外的 font-weight 值在Android上的React-Native中没有效果
- 不支持`100vh`设置
- 其他不支持的样式

`React-Native`的样式基于开源的跨平台布局引擎 [Yoga](https://github.com/facebook/yoga) ，样式基本上是实现了 CSS 的一个子集，所以会有些样式不支持，在出错时会给出支持的样式列表。如 RN 不支持针对某一边设置 style，即 border-bottom-style 会报错。

- 必须使用 Flex 布局
- 文字要包在 `Text` 组件里面，否则不显示
- `position:fixed` React-Native 不支持
- 样式选择器仅支持类选择器，且不支持 **组合器**
- RN 中 View 标签默认主轴方向是 column

如果需要兼容各端，最好在布局中显式声明主轴方向。

### 2.3.3 其他问题

- 不支持同步的 Storage 存取

即不支持`Taro.setStorageSync`和`Taro.getStorageSync`两个同步方法，因此需要对已有代码进行重构，使用`async/await`和`Promise`进行处理。

- 找不到方法/函数

所有的方法参数传递，需要`bind(this)`

- this.setState is not a function

寻找未添加`bind(this)`的方法参数传递，添加`bind(this)`

- `componentWillMount()`即将废弃

使用`componentDidMount()`实现

- 没有读取文件并进行 base64 编码

使用`rn-fetch-blob`库解决。

### 2.3.4 其他参考

Taro 官方提供了[各端开发前注意事项](https://taro-docs.jd.com/taro/docs/before-dev-remind.html)，特别是使用 RN 的样式区别。

Taro 官方提供了[最佳实践](https://taro-docs.jd.com/taro/docs/best-practice.html)，可以参考。

Taro 官方提供了[更多资源](https://taro-docs.jd.com/taro/docs/composition.html)，可以查看其他团队的文章分享和遇到的问题解决。

## 2.4 热更新方案

目前业界 RN 热更新方案同样分为 全量更新 和 增量更新。原理同 Weex 类似，此处不再赘述。

全量更新分为两部分，即`Bundle`文件和资源的更新。

- 目前方案

当前项目页面较少，整个 Bundle 文件不到 2MB，目前使用全量更新。

资源文件夹先压缩再上传；`App`端下载后进行`md5`验证，验证通过则解压到缓存目录。

在下次重新打开 App 才读取缓存的`Bundle`和 资源。

接口格式可参考以下方案：

```
{
    "error_code": 0,
    "error_message": "成功",
    "data": {
        "bundle": {
            "version": "190108174500",
            "url": "https://xxx/main.jsbundle",
            "md5": "xxx"
        },
        "assets": {
            "version": "190108174500",
            "url": "https://xxx/assets.zip",
            "md5": "xxx"
        }
    }
}
```

因为该项目的用户是公司内部员工，目前第一版，所以没有对复杂情况进行处理，后续应用后再持续设计。

如支持不同设备或不同`App`版本使用不同的`Bundle`等等。

- Pushy

目前官方推荐热更新框架[Pushy](https://pushy.reactnative.cn)，可以了解下，原理都是差不多的，实现会更方便。

## 2.5 相关命令及其作用

- Taro 转 RN

```
taro build --type rn
```

- 调试代码

```
taro build --type rn --watch
```

- 打包 RN Bundle 和资源

```
cd rn_temp && node ../node_modules/react-native/local-cli/cli.js bundle --entry-file ./rn_temp/index.js --bundle-output ./bundle/main.jsbundle --assets-dest ./bundle --dev false
```

执行 Taro 转 RN 命令后，在项目中会生成 rn_temp 文件夹，该文件夹即完整的 RN 代码，打包 RN 需要进入该目录执行。

打包后会生成 Bundle 文件和资源文件夹，用于替换原生保存的 RN 内容。

# 三、总结

- 友好度——转换工作复杂度

由于作者是开发`iOS`客户端的，且于4年前已经在现有项目中集成过`React-Native`页面，对`React-Native`的原理和机制有过研究，实践过相关技术栈，所以转换工作相对容易。

对于不是客户端出身，或者不熟悉`React-Native`/`Weex`等客户端跨平台框架的机制和技术栈的同学，需要做点其他功课。

- 多端适配以`React-Native`样式为主

如果需要适配多端，需要以 `React-Native` 的约束来管理样式。

因为样式上`H5`最为灵活，小程序次之，`React-Native`最弱（`React-Native`的样式基于开源的跨平台布局引擎 [Yoga](https://github.com/facebook/yoga) ，样式基本上是实现了 CSS 的一个子集），统一多端样式即是对齐短板，也就是要以 `React-Native` 的约束来管理样式，同时兼顾小程序的限制。

- `Taro`-> `React-Native` -> `iOS/Android`链条长

通过 `Taro` 转`React-Native`，`React-Native`再转`iOS/Android`，链条过长，会面临复杂度增加、维护成本较大、组件适配、客户端系统版本和语言升级维护等问题，同时影响一些复杂功能的实现方案的决策和估时。

毫无疑问，这是复杂度非常高的架构设计，对开发人员的能力和各方面经验有较高要求。

为了尽早覆盖跨客户端，`Taro`官方选择 `React-Native`无疑是最快速和易行的方案。

- 跨端效果和质量

总体而言，在功能不复杂的项目中，进行`Taro`转换`React-Native`，虽然修改较多但都不算难处理，虽有几处转换问题需等官方处理，但转换后的效果和质量还算OK。



-END-
欢迎到我的博客交流：[https://zackzheng.info](https://zackzheng.info)