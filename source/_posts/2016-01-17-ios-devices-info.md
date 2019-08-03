---
title: iOS所有设备的分辨率、尺寸和缩放因子，放大模式区别和6P实际分辨率
date: 2016-01-17 16:00:00
tags: 
     - iOS
categories: iOS
keywords: iOS
description: 记录iOS所有设备的分辨率、尺寸，和一些细小的注意点等
---

偶尔需要查下设备的分辨率和缩放因子，经常用的时候才网上搜下确认下，比较麻烦。现在整理下方便自己和别人以后使用。

*   1 inch = 2.54cm = 25.4mm
*   @2x即1个Point被渲染成1个2x2的像素矩阵，即某个方向上1pt存在2px
*   PPI（Pixel Per Inch by diagonal）：沿着对角线每英寸所拥有的像素数目。如iPhone 6的PPI=![gif.latex.gif](304530-f1e621f5f6cd6666.gif)

# iPhone 设备

*   放大模式下，iPhone 6分辨率为640x1136(Upsampling，插值采样)，iPhone 6 Plus/6S Plus分辨率为1125×2001(Downsampling，缩减像素采样)。
*   iPhone 6 Plus/6S Plus的实际分辨率是1920x1080，实际缩放为@2.46x。（也就是说实际上应该用@2.46x的素材。iOS 拿到@3x绘制结果，实时地再缩小到实际的1080 x 1920分辨率，Downsampling）详见[知乎上的讨论](http://www.zhihu.com/question/25288571)。

| 设备                  | 尺寸(inch) | 逻辑分辨率(pt) | 屏幕分辨率(px) | Scale Factor |
| --------------------- | ---------- | -------------- | -------------- | ------------ |
| iPhone                | 3.5        | 320x480        | 320x480        | @1x          |
| iPhone 3G/3GS         | 3.5        | 320x480        | 320x480        | @1x          |
| iPhone 4/4S           | 3.5        | 320x480        | 640x960        | @2x          |
| iPhone 5/5S/5C        | 4          | 320x568        | 640x1136       | @2x          |
| iPhone 6/6S           | 4.7        | 375x667        | 750x1334       | @2x          |
| iPhone 6 Plus/6S Plus | 5.5        | 414x736        | 1242x2208      | @3x          |
| iPhone SE             | 4          | 320x568        | 640x1136       | @2x          |
| iPhone 7/8            | 4.7        | 375x667        | 750x1334       | @2x          |
| iPhone 7 Plus/8 Plus  | 5.5        | 414x736        | 1242x2208      | @3x          |
| iPhone X              | 5.8        | 375x812        | 1125x2436      | @3x          |

.

# iPad 设备

| 设备                                 | 尺寸(inch) | 逻辑分辨率(pt) | 屏幕分辨率(px) | Scale Factor |
| ------------------------------------ | ---------- | -------------- | -------------- | ------------ |
| iPad                                 | 9.7        | 1024x768       | 1024x768       | @1x          |
| iPad 2                               | 9.7        | 1024x768       | 1024x768       | @1x          |
| The New iPad                         | 9.7        | 1024x768       | 2048x1536      | @2x          |
| iPad 4                               | 9.7        | 1024x768       | 2048x1536      | @2x          |
| iPad Air                             | 9.7        | 1024x768       | 2048x1536      | @2x          |
| iPad Air 2                           | 9.7        | 1024x768       | 2048x1536      | @2x          |
| iPad Pro (12.9-inch)                 | 12.9       | 1366x1024      | 2732x2048      | @2x          |
| iPad Pro (9.7-inch)                  | 9.7        | 1024x768       | 2048x1536      | @2x          |
| iPad (5th generation)                | 12.9       | 1024x768       | 2048x1536      | @2x          |
| iPad Pro (12.9-inch, 2nd generation) | 12.9       | 1366x1024      | 2732x2048      | @2x          |
| iPad Pro (10.5-inch)                 | 10.5       | 1112x834       | 2224x1668      | @2x          |

.

# iPad mini设备

| 设备        | 尺寸(inch) | 逻辑分辨率(pt) | 屏幕分辨率(px) | Scale Factor |
| ----------- | ---------- | -------------- | -------------- | ------------ |
| iPad Mini   | 7.9        | 1024x768       | 1024x768       | @1x          |
| iPad Mini 2 | 7.9        | 1024x768       | 2048x1536      | @2x          |
| iPad Mini 3 | 7.9        | 1024x768       | 2048x1536      | @2x          |
| iPad Mini 4 | 7.9        | 1024x768       | 2048x1536      | @2x          |

.

# iPod Touch 设备

| 设备         | 尺寸(inch) | 逻辑分辨率(pt) | 屏幕分辨率(px) | Scale Factor |
| ------------ | ---------- | -------------- | -------------- | ------------ |
| iPod Touch 1 | 3.5        | 320x480        | 320x480        | @1x          |
| iPod Touch 2 | 3.5        | 320x480        | 320x480        | @1x          |
| iPod Touch 3 | 3.5        | 320x480        | 320x480        | @1x          |
| iPod Touch 4 | 3.5        | 320x480        | 640x960        | @2x          |
| iPod Touch 5 | 4          | 320x568        | 640x1136       | @2x          |
| iPod Touch 6 | 4          | 320x568        | 640x1136       | @2x          |

.



-END-
欢迎到我的博客交流：https://zackzheng.info