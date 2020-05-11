---
title: Taro图片素材不显示？--Taro下的webpack配置
date: 2020-05-11 19:00:00
tags: 
     - Taro
     - webpack
categories: Tech
description: Taro下的webpack配置，对图片素材的base64编码的处理
---

# 前言

最近遇到`Taro`下图片素材没显示的小问题，解决完顺藤摸瓜了解了下前端领域`webpack`及其`Loader`的一些原理。

本文只是简单作个记录，涉及`Taro`下一些`webpack`的配置，可以为刚好遇到类似问题的人提供可能的助益。

文末也简单讨论了图片素材转变成`base64`编码这种处理方式。

# 场景和问题解决

使用 `map`组件展示`markers`，代码如下：

```typescript
import LocationIcon from '../xxx.png'
class XXX {
  render () {
    // 定义marker
    var markers = [{
        iconPath: LocationIcon,
        id: 0,
        latitude,
        longitude,
        width: 38,
        height: 50
    }];
    return (
      ...
      <Map className='...' id="..." longitude={longitude} latitude={latitude} markers={markers}></Map>
      ...
    )
  }
}
```

## 问题一：真机没有显示 iconPath 的素材？

**思路：**

- 可能 `iconPath`的设置有限制，查看 [微信小程序Map组件文档](https://developers.weixin.qq.com/miniprogram/dev/component/map.html#map)。

> `iconPath`，显示的图标，类型`string`，项目目录下的图片路径，支持网络路径、本地路径、代码包路径（[2.3.0]）

图片是按 [Taro-引用图片、音频、字体等文件](https://taro-docs.jd.com/taro/docs/static-reference.html#引用图片-音频-字体等文件) 所说，通过 `ES6` 的 `import` 语法来引用的，即“项目目录下的图片路径”，所以不是组件使用问题。

+ 检查运行过程中生成的Wxml，看看是否生成有问题，以此倒推

```jsx
<Map className='...' id="..." longitude={...} latitude={...} markers="[{"iconPath":"data:image/png;base64,iVBOR..."}]"></Map>
```

发现`iconPath`的值变成了`base64`编码后的数据。

**小结：**

至此，找到直接原因，真机环境`Taro`打包对有些素材进行base64编码，设置给`iconPath`后，不满足文档要求，所以无法显示。



## 问题二：那为什么有些素材有展示？有些没有展示？

**思路：**

+ 查看几处引用素材的地方运行过程中生成的Wxml：

```jsx
<image src="data:image/png;base64,iVBOR..." ...>
<image src=".../.../xxx.png" ...>
```

发现相同的用法，有些素材是变成`base64`数据，有些依旧是素材路径。

+ 比较素材的区别

查看素材发现，编码成`base64`的，都是几KB的小素材；使用路径的，是十几KB甚至更大的素材。

**小结：**

依此推断，`Taro`只对小素材进行了`base64`编码处理，而对于较大的素材，没有改动。



## 问题三：这个处理是如何配置的？

**思路：**

大晚上的，私聊了一下`Taro`的主程，告知这是`webPack`进行的处理。于是找到了Taro文档 [进阶指南-编译配置详情](https://taro-docs.jd.com/taro/docs/config-detail.html#miniimageurlloaderoption)。此处对配置进行了简单说明。

```
mini.imageUrlLoaderOption
```

需要参考[`url-loader README`](https://github.com/webpack-contrib/url-loader)关于配置字段的说明。

最终配置如下：

```js
// config/index.js
mini: {
    imageUrlLoaderOption: {
      limit: 10000,
      mimetype: 'image/png',
      encoding: 'base64'
    },
    ...
}
```

此处配置`imageUrlLoaderOption`的作用是，对于`png`的图片素材，如果体积超过了10000B，则进行`base64`编码。

`Taro`在新版本完全开放了`WebPack`的配置，有很多插件可以配置，文档列的比较详细，此处就不再赘述了。主要是以后遇到一些问题可以想到用`webpack`这个层面去解决。

# 扩展

到此可能会想，为什么会有这样的配置？有什么好处？

简单总结下我的想法：

1. 相比网络图片，使用`base64`编码的方式，减少加载图片的`http`请求次数，没有因网络加载而带来的延时。
2. `base64`编码只是一种编码方式，不是一种压缩方式，它实际上会增加图片本身的大小（在我之前的文章有介绍过），图片越大增加的越多。
3. 相比网络图片，如果是大图进行`base64`编码，和多一次`http`请求的耗时比起来，显然划不来。

疑问：

1. 同一个素材多处使用，进行`base64`编码，会不会导致`CSS`大小增加？不清楚有待考证。
2. 小的本地素材，进行`base64`编码有什么好处？大概同“素材加载”和“CSS加载”的区别有关系。

如果有前端的同学知道，请不吝赐教哈。



-END-
欢迎到我的博客交流：https://zackzheng.info