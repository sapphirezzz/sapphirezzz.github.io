---
title: App图片压缩裁剪原理和上传方案，以及那些有趣的事儿...
date: 2016-02-26 16:00:00
tags: 
     - iOS
     - 图片压缩
categories: Tech
keywords: iOS Android 图片压缩
description: App图片压缩裁剪原理和上传方案，以及那些有趣的事儿...
---

# 目录

- App怎么压缩质量？
  - iOS和Android压缩接口
  - 实验一
- 如何计算图片的大小？
- JPEG&JFIF压缩做了什么？
  - 色彩空间转换
  - 缩减取样
  - 离散余弦变换
  - 量化
  - 编码
- App怎么裁剪分辨率？
- 图片压缩裁剪上传方案
  - 实验一：上传速度
  - 实验二：用户设备主要分辨率
  - 实验三：上传的照片的主要分辨率
  - 实验四：压缩质量的大致规律
  - 实验五：等比例裁剪后压缩质量的大致规律
  - 实验六：WebP压缩
  - 制定方案
- 总结
- 附录
  - 图像的一些概念澄清
  - 压缩算法概念
  - 微信图片处理规律
  - Base64编码后大小
  - iOS的pt与Android的sp
  - Android图片质量会比iPhone的差？
  - iOS的UIImage保存图片问题
  - 压缩再压缩做了什么

最近有反馈说App上传图片偶尔会失败，特别是在网速慢和iPhone 6s的机器上。有些提示是“413 Request Entity Too Large”，Request大小1.15MB。之前只是简单地压到0.7，没有做裁剪等其他处理。

所以这几天整体研究下iOS和Android的图片压缩裁剪和JPEG压缩原理，也稍微找了微信发图片的规律，越深入发现越多有趣的东西，问题一环扣一环。

# App怎么压缩质量？
最先接触的是压缩质量，所以看下压缩质量做了什么。

## iOS和Android压缩接口
iOS:

``` 
UIImageJPEGRepresentation(UIImage * __nonnull image, CGFloat compressionQuality);  
// return image as JPEG. May return nil if image has no CGImageRef or invalid bitmap format. compression is 0(most)..1(least)
```

Android:

``` 
/** 
    * Write a compressed version of the bitmap to the specified outputstream. 
    * If this returns true, the bitmap can be reconstructed by passing a 
    * corresponding inputstream to BitmapFactory.decodeStream(). Note: not 
    * all Formats support all bitmap configs directly, so it is possible that 
    * the returned bitmap from BitmapFactory could be in a different bitdepth, 
    * and/or may have lost per-pixel alpha (e.g. JPEG only supports opaque 
    * pixels). 
    * 
    * @param format   The format of the compressed image 
    * @param quality  Hint to the compressor, 0-100. 0 meaning compress for 
    *                 small size, 100 meaning compress for max quality. Some 
    *                 formats, like PNG which is lossless, will ignore the 
    *                 quality setting 
    * @param stream   The outputstream to write the compressed data. 
    * @return true if successfully compressed to the specified stream. 
*/
compress(CompressFormat format, int quality, OutputStream stream)
```

从官方注释看，iOS的compressionQuality取值从0-1，且“1”注明是least，也就是说**“1”不是不压缩，而是压缩强度最弱**；Android的quality取值0-100，也是**100代表最大质量，不是不压缩**。最后面测试也验证了这个问题，而*不是网上很多文档说的不压缩*。我们先统称compressionQuality和quality为质量系数，下文也是。

## 实验一

于是拍了一张照片A，然后Android和iOS都压缩得到照片B。

对比了A、B两张图，想看看到底影响了什么：

> 宽度：2448像素
> 高度：3264像素
> 分辨率：72像素/英寸
> 文档：22.9MB
> 通道：3（RGB）

注：*需弄明白附录的关于图像的一些概念澄清*。

两张照片看到的参数都一样，那为什么它们的文件大小会不一样呢？（*过程中发现iOS的UIImage会做多一些工作，使得实验结果有误，原因后面会提到。*）（用Picasa看图软件可以查看质量，数值上看到等于我们的质量系数）所以就想到第二个问题，文件大小怎么计算的？由什么决定？

# 如何计算图片的大小？
总分辨率 * 像素表示的位数。

像素表示的位数：这就涉及到色彩模式，比较常见有RGB、CMYK、YUV等。RGB一般用RGB24（还有RGB555、RGB565、RGB32），即红绿蓝都分别用8位表示，所以用了24位表示一个像素，可以组合出2^24种颜色。

上面两张图水平有2448个像素，垂直有3264个像素，每个像素用24b表示，按这公式大小应该都是：2448x3264x24b＝23970816B=22.86MB。为什么呢？

后来明白，原来这公式是算位图的占用空间大小，而JPEG&JFIF是将位图压缩，不仅压缩图像质量还压缩图像占用空间（后面会讲到）。也就是说**图像压缩不等于压缩质量和分辨率，还有压缩占用空间**。

网上查到JFIF文件没有计算大小的公式，因为压缩质量和压缩后大小没有特定关系，如线性关系。**那么JPEG&JFIF压缩做了什么？这个质量到底代表了什么？**

# JPEG&JFIF压缩做了什么？

其实JPEG&JFIF做了两件事情：

1. 去掉视觉上的冗余信息
2. 去掉数据本身结构的冗余

第一步实现通过色彩空间转换、缩减取样、离散余弦变换、量化，第二步实现通过编码。

其实这部分可以选择跳过，只是我为了理解压缩质量是怎么体现的而去看的，后面也发现理解后很多问题都很清晰明白。

## 色彩空间转换
JPEG需要YUV色彩模式，所以需要将RGB转成YUV：
- Y=0.299R'+0.587G'+0.114B'
- U=-0.147R'-0.289G'+0.436B'
- V=0.615R'-0.515G'-0.100B'

## 缩减取样
YUV分别代表亮度、色度、饱和度，因为人类的眼睛对于亮度差异的敏感度高于色彩变化，所以一般会对U、V进行缩减采样。在JPEG上这种缩减取样的比例可以是4:4:4（无缩减取样）、4:2:2、4:2:0。所以经常会看到YUV444，YUV422和YUV420等。

## 离散余弦变换
将每个8x8的子区域转换到频率空间，这部分是无损的。

-415 -30 -61  27  56 -20 -2  0
   4    -22 -61  10  13  -7  -9  5
  -47   7   77  -25 -29 10  5  -6
  -49  12  34  -15 -10  6   2   2
  12   -7  -13   -4   -2   2   -3  3
   -8    3    2     -6   -2   1   4   2
  -1     0    0    -2    -1   -3  4  -1
   0     0    -1   -4    -1    0   1  2

## 量化
量化是有损的过程，也是失真的主要原因。上面矩阵中每个值都是幅度，量化是利用人眼特点在高频率上降低信息的数量（简单地把频率领域上每个成分，除以一个对于该成分的常数就可完成，且接着舍位取最接近的整数），与一个基本矩阵运算然后得到一个新的矩阵，**该新矩阵数据基本集中在左上角**。

其实**量化做的就是减少非0系数幅度和增加0值系数的个数**。
**我们接口中的质量系数就是和这里的基本矩阵有关。**

## 编码

Z扫描0值行程长度编码、哈夫曼编码。
Z扫描0值行程长度编码利用了量化后矩阵的特点，使得0值的都能串在最后面，对于过早结束的最后用EOB表示。
这一步就是**压缩文件的占用空间**。

# App怎么裁剪分辨率？
弄清楚压缩质量问题后，我们知道，影响位图的大小有分辨率，那么减少分辨率也就能使压缩得更小了。注意这里裁剪分辨率不等于裁剪图片，不会丢失图片的某一部分。

iOS:

``` 
UIGraphicsBeginImageContext(newSize);
[imageFixed drawInRect:CGRectMake(0, 0, newSize.width, newSize.height)];
UIImage *scaledImage = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();
```

Android:

``` 
Bitmap image = BitmapFactory.decodeStream(file);
int bitmapWidth = image.getWidth();
int bitmapHeight = image.getHeight();
Matrix matrix = new Matrix();
matrix.postScale(scaleRatio, scaleRatio);
Bitmap scaledBitmap = Bitmap.createBitmap(image, 0, 0, bitmapWidth, bitmapHeight, matrix, false);
```

我们是等比例裁剪，比例和裁剪后占用空间大小并不一定成线性关系，这个裁剪具体怎么实现的和每个像素的质量有关系。

# 图片压缩裁剪上传方案
明白一个图片的清晰度等受什么影响之后，可以来定下方案了，但是还得做些测试获取数据，依据数据结果来定方案。

## 实验一：上传速度
我们用Charles模拟了各种网络环境下上传相同照片的网速：

|   环境   |  wifi  |  8M   |  16M   |  32M   |
| :----: | :----: | :---: | :----: |
| 速度KB/s | 754.78 | 115.4 | 115.78 | 116.25 |

|   环境   |  32M光纤  | 100M光纤 |   3G   |   4G   |
| :----: | :----: | :---: | :----: |
| 速度KB/s | 174.65 | 167.33 | 114.18 | 174.73 |

这些不等价于我们用户的网速平均值，而且只取一次样本，所以只是作为参考。

## 实验二：用户设备主要分辨率
因为照片都是在用户设备上看的，所以了解下用户的主流分辨率，将分辨率裁剪到趋近或稍大于该主流分辨率会比较适合。iOS和Android都取接近的值。

在友盟查看了我们的应用的相关数据：

- Android，较大占比的是宽度1080像素和720像素的设备。宽度1080像素的，新增用户占比49.69%，启动次数占比44.29%；宽度720像素的，新增用户占比32.47%，启动次数占比37.76%。
- iOS，较大占比的是宽度1242像素、750像素和640像素的设备。宽度1242像素的，新增用户占比31.84%，启动次数占比26.91%；宽度750像素的，新增用户占比36.20%，启动次数占比31.59%；宽度640像素的，新增用户占比20.36%，启动次数占比20.84%。

## 实验三：上传的照片的主要分辨率
找下上传的照片的主要分辨率，可以针对这些分辨率去确定和衡量方案。
也是在友盟上看我们的应用的数据：

- iOS主流机型是iPhone6、iPhone6s、iPhone6 Plus、iPhone6s Plus，占到70%左右。它们的拍照分辨率分别是2448x3264，3024x4032。
- Android相对较碎片化，只能是针对最大的分辨率去衡量。

## 实验四：压缩质量的大致规律

2448x3264照片压缩后的数据大小（单位B）：

| 压缩强度 |   图片1   |   图片2   |   图片3   |   图片4   |
| :--: | :-----: | :-----: | :-----: | :-----: |
| 0.1  | 197777  | 174464  | 152897  | 195214  |
| 0.2  | 212779  | 183532  | 163225  | 209426  |
| 0.3  | 253658  | 205512  | 196188  | 248320  |
| 0.4  | 360091  | 256013  | 283607  | 349968  |
| 0.5  | 498199  | 336823  | 390525  | 485286  |
| 0.6  | 724491  | 488737  | 548437  | 698421  |
| 0.7  | 1209508 | 883664  | 936968  | 1161342 |
| 0.8  | 1475669 | 1063724 | 1130241 | 1415084 |
| 0.9  | 1700461 | 1247600 | 1334143 | 1636293 |
| 1.0  | 4568157 | 3629379 | 4139209 | 4433556 |
| 原图占用 | 1714688 | 1257647 | 1353635 | 1647936 |

3024x4032照片压缩后的数据大小（单位B）：

| 压缩强度 |   图片5   |   图片6   |   图片7   |   图片8   |
| :--: | :-----: | :-----: | :-----: | :-----: |
| 0.1  | 551527  | 325434  | 296792  | 492494  |
| 0.2  | 619307  | 352608  | 328183  | 547533  |
| 0.3  | 785915  | 422342  | 416459  | 683265  |
| 0.4  | 1061504 | 537826  | 589206  | 931686  |
| 0.5  | 1408297 | 891288  | 839938  | 1290788 |
| 0.6  | 2071014 | 973894  | 1108102 | 1607344 |
| 0.7  | 2547536 | 1228021 | 1756824 | 2479774 |
| 0.8  | 2652258 | 1320258 | 1831889 | 2617015 |
| 0.9  | 3336839 | 1603754 | 2212214 | 3079872 |
| 1.0  | 7282230 | 4337074 | 5787540 | 7962849 |
| 原图占用 | 2635183 | 1511290 | 1860418 | 2641699 |

- 会出现压缩出来反而比原图大的问题，后面会讨论
- 在没有裁剪的情况下，压到0.6依旧不是太理想，特别是分辨率更高的照片。
- 在excel表格中将数据组成折线图，可以看出在0.9、0.8的时候下降幅度较大，后面相对平缓一点。

## 实验五：等比例裁剪后压缩质量的大致规律
2448x3264照片等比例压缩到宽度1224像素时的数据大小（单位B）：

| 压缩强度 |   图片1   |   图片2   |   图片3   |   图片4   |
| :--: | :-----: | :-----: | :-----: | :-----: |
| 0.1  |  58975  |  52025  |  43301  |  60029  |
| 0.2  |  64446  |  55583  |  46529  |  65347  |
| 0.3  |  76982  |  63805  |  55494  |  77891  |
| 0.4  |  99499  |  77605  |  76695  | 100042  |
| 0.5  | 129826  |  95116  | 103949  | 129597  |
| 0.6  | 173036  | 120654  | 144188  | 173569  |
| 0.7  | 246847  | 174977  | 209876  | 249223  |
| 0.8  | 303546  | 208392  | 261322  | 308841  |
| 0.9  | 411181  | 294795  | 358003  | 420013  |
| 1.0  | 1574657 | 1280563 | 1536779 | 1566585 |
| 原图占用 | 1714688 | 1257647 | 1353635 | 1647936 |

3024x4032照片等比例压缩到宽度1224像素时的数据大小（单位B）：

| 压缩强度 |   图片5   |   图片6   |   图片7   |   图片8   |
| :--: | :-----: | :-----: | :-----: | :-----: |
| 0.1  | 111697  |  70802  |  54321  | 121994  |
| 0.2  | 127179  |  77524  |  61810  | 136869  |
| 0.3  | 163959  |  94048  |  79713  | 172194  |
| 0.4  | 225870  | 122258  | 113295  | 230197  |
| 0.5  | 302097  | 157160  | 155152  | 300772  |
| 0.6  | 395440  | 202312  | 210836  | 385183  |
| 0.7  | 524345  | 273895  | 295530  | 505417  |
| 0.8  | 623839  | 323884  | 363596  | 593923  |
| 0.9  | 777293  | 420943  | 481205  | 745946  |
| 1.0  | 2184791 | 1530139 | 1774130 | 2291622 |
| 原图占用 | 2635183 | 1511290 | 1860418 | 2641699 |

- 可以看到裁剪这个分辨率后压缩质量0.6的大小相对可接受，而且图片质量影响也较小。
- 在这个分辨率下，压缩质量0.8时基本压缩到1/4至1/5。

## 实验六：WebP压缩
WebP据称在同等质量下大小可以压缩至JPEG的2/3。在iOS和Android都做了WebP的压缩测试，发现压缩速度非常慢，需要20s-35s，这是很不可接受的。上网查到WebP的压缩效率的确较慢，是JPEG的8倍左右，而且相比JPEG需要耗费更多的系统性能。所以本来想用WebP做兜底的方案暂时落空。

## 制定方案

方案的目的是：

1. 上传图片不超时
2. 处理后质量清晰度可接受
3. 缩短上传耗时

结合实验结论：

- 根据实验一，平均上传速度100KB/s，另外我们测试很差情况下有15KB/s的情况。我们的接口超时30s，所以可接受的图片大小最大为450KB。
- 根据实验二，等比例宽度取750-1242像素比较合适。
- 根据实验三、四、五，等比例裁剪到宽度1224像素后压缩至0.6、0.7大小相对可接受。质量相对可接受（主观感受）。
- [JPG、PNG和GIF图片的基本原理及优化方法](http://www.mahaixiang.cn/Photoshop/400.html)讲到，通常建议JPG质量最好是在60左右的原因。当在Photoshop中把质量设置低于51的时候，它就会执行另一个叫做“降色采样”的优化算法，它会在8个像素周围平均采样，这样会在边缘产生杂色。

定下方案：

1. 照片宽度大于1224像素（因为iPhone6照片宽度2448所以想取个可以整除的）时等比例裁剪宽度成1224。因为分辨率太大甚至两倍于手机分辨率实际没有任何用处。
2. 压缩质量系数至0.8(80)看下大小是否小于300KB，排除一些小分辨率的照片。小于则上传，大于则继续压缩，取0.7(70)，排除一些中等的，如果还大于则取0.6(60)后不判断直接上传。根据上面的实验可以看到在宽度1224像素下基本都会小于300KB，大于的则处于450KB内，且450KB是网速最差的情况，因此基本可以保证上传。

# 总结
方案目前看压缩的质量和时间控制都相对较好，但是还要继续观察一段时间看需不需要再调整参数。在这过程中理清了很多概念，了解了图片压缩过程中发生了什么事情，挺有趣的，根本停不下来。

# 附录
文中涉及到的一些概念以及发现的一些其他相关事情。

## 图像的一些概念澄清

其实图像真正的信息是它的**总分辨率**（图像宽度x图像高度）和**分辨率**（水平分辨率&垂直分辨率），**而涉及到英寸、厘米等长度单位时的大小，其实不是图像的信息，只是在涉及到外部，比如打印、设备屏幕显示等等时，换算出来的。**

- 水平分辨率&垂直分辨率

分辨率一般单位ppi（也有dpi），即每英寸上多少个像素。水平分辨率即水平方向上像素个数/水平长度，垂直分辨率同理。**一般水平分辨率和垂直分辨率是相等的，所以日常也简称为分辨率。**dpi即每英寸上多少点。这两个单位根据不同显示设备有换算关系。

- 图像宽度&图像高度

其实标准些，讲宽度&高度时一般是讲图像水平/垂直上有多少个像素。如一张2448x3264的照片，则宽度是2448像素，高度是3264像素。也有讲宽度是86.36厘米，高度是115.15厘米，一般在打印照片等情况下。

- 分辨率

日常讲分辨率时其实可以指很多情况，一般指图像宽度x高度，如2448x3264，指水平上有2448个像素，垂直上有3264个像素。

## 压缩算法概念

- JPEG

一个名称为Joint Photographic Experts Group的组织，也是一种压缩算法。JPEG本身只有描述如何将一个视频转换为字节的数据流（streaming），但**并没有说明这些字节如何在任何特定的存储媒体上被封存起来**。JPEG有有损压缩和无损压缩，无损压缩的没有得到什么支持，所以一般讲JPEG指它的有损压缩。

- JFIF

JPEG File Interchange Format，JPEG文件交换格式，**详细说明如何从一个JPEG流，产出一个适合于电脑存储和传输的文件**。一般后缀有.jpeg、.jpg、.jfif以及.jif。

- Bitmap

又称栅格图，是使用像素阵列来表示的图像。非压缩格式，从左往右从上往下扫描，占用较大存储空间。

> https://zh.wikipedia.org/wiki/JPEG
> https://zh.wikipedia.org/wiki/%E4%BD%8D%E5%9B%BE

## 微信图片处理规律

经过简单测试发现，在微信，A发一张原图给B时，B直接保存缩略图：
- iOS版的微信保存下来是等比例压缩高度为800，质量80。
- Android版的微信保存下来是等比例压缩高度为1280，质量80。

朋友圈的图片保存下来，都是等比例到高1280像素，质量和大小还没找到规律。
为什么微信是取高固定而不是宽呢？可以思考下。

## Base64编码后大小

我们现在还用着Base64编码图像数据，后面需改成提交表单方式。

Base64要求把每三个8Bit的字节转换为四个6Bit的字节（3*8 = 4*6 = 24），然后把6Bit再添两位高位0，组成四个8Bit的字节，也就是说，**转换后的字符串理论上将要比原来的长1/3**。

## iOS的pt与Android的sp
我在[iOS所有设备的分辨率、尺寸和缩放因子，放大模式区别和6P实际分辨率](https://sapphirezzz.github.io/blog/2016/01/17/ios-devices-info/)文章里讲了iOS设备的pt和px关系。
iOS的pt是开发单位，一个逻辑point对应一个点dot。@1x设备上一个pt上显示1个px（像素），@2x上一个pt显示2个px，@3x上显示3个px。平时在storyboard上设置时指的都是pt。

Android的sp一种基于屏幕密度的抽象单位。

## Android图片质量会比iPhone的差？

在[为什么Android的图片质量会比iPhone的差？](http://www.cnblogs.com/MaxIE/p/3951294.html)、[[Android算法] 【04/28 bug修改】android图片压缩终极解决方案](http://www.eoeandroid.com/thread-570303-1-3.html?_dsign=bc58b99e)、[Android图片编码机制深度解析（Bitmap，Skia，libJpeg）](http://www.cnblogs.com/hrlnw/p/4403334.html)三篇文章中讲了Android系统在压缩上的一些不为人知的问题。

大致是，Android编码保存图片就是通过Java层函数——Native层函数——Skia库函数——对应第三方库函数（例如libjpeg），这一层层调用做到的。 libjpeg在压缩图像时，有一个参数叫optimize_coding，如果设置optimize_coding为TRUE，将会使得压缩图像过程中基于图像数据计算哈弗曼表，由于这个计算会显著消耗空间和时间，默认值被设置为FALSE。对于当时的计算设备来说，空间和时间的消耗可能是显著的，但到今天，这似乎不应再是问题。但谷歌的Skia项目工程师们对optimize_coding在Skia中默认的等于了FALSE，这就意味着更差的图片质量和更大的图片文件。还有其他和iOS的比较可以看下。

也讲到了Android可以替换libjpeg库达到设置为TRUE的目的。

## iOS的UIImage保存图片问题

起初发现压缩后保存图片，不同压缩质量系数，得出来的文件大小趋势和计算出的大小趋势不同，所以怀疑使用NSData初始化UIImage时多做了什么。

``` 
NSData *imageData06 = UIImageJPEGRepresentation(scaledImage, 0.6);
UIImage *image06 = [UIImage imageWithData:imageData06];
UIImageWriteToSavedPhotosAlbum(image06, nil, nil, nil);
```

通过软件发现，不同压缩质量系数，得出来的NSData保存成UIImage图片，看到的质量都是92，按道理应该是对应的质量系数才对。于是想将NSData保存成文件到目录，读取出文件大小。

``` 
NSString *documents = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0];
NSString *filePath = [documents stringByAppendingPathComponent:@"image.jpg"];
NSData *imageData = UIImageJPEGRepresentation(image, 0.7);
NSError *error = nil;
[imageData writeToFile:filePath options:NSDataWritingAtomic error:&error];
if (error != nil) {
	NSLog(@"error = %@", error);
}else {
	NSLog(@"success");
}
NSLog(@"imageData = %u", (unsigned)imageData.length);
NSFileManager* manager = [NSFileManager defaultManager];
if ([manager fileExistsAtPath:filePath]){
	unsigned long long size = [[manager attributesOfItemAtPath:filePath error:nil] fileSize];
	NSLog(@"0.7后文件大小 %llu", size);
}
UIImage *imageFromData = [UIImage imageWithData:imageData];
UIImageWriteToSavedPhotosAlbum(imageFromData, nil, nil, nil);
UIImage *imageRead = [[UIImage alloc] initWithContentsOfFile:filePath];
UIImageWriteToSavedPhotosAlbum(imageRead, nil, nil, nil);
```

读取文件大小size和NSData大小imageData.length打印一致，而与imageRead、imageFromData的大小不一样，因此可以证明UIImage这个对象本身还做了其他事情。

在[AFNetworking2.0源码解析](http://www.udpwork.com/item/13533.html)这篇文章的截图显示，UIImage的imageWithData方法堆栈显示还调用了哈夫曼解码。

## 压缩再压缩做了什么

1. 首先谈下看图软件怎么展示图片。打开.jpg图片时，看图软件将文件转换成位图才显示出来，即把量化表矩阵与基本转化矩阵运算得出图片当前的位图（因为经过了压缩，所以该位图与原位图不同）。
2. 压缩后再压缩也是，先将图片变回位图，再将位图按现在的压缩质量系数压缩。
3. 所以压缩后的图片有可能变大，我猜测原因是，压缩前的占用空间压缩率较大，而再压缩时选的质量系数较大，导致压缩占用空间率（即压缩的第二步）较小，所以导致质量其实是变差了，但是占用空间反而可能变大。

变大的例子如：本来图片A大小500KB，压缩质量系数选择0.7，得到B图片90KB，再拿这张B图片去压，压缩质量系数1.0，得出C图片。前面我们说压缩不仅压缩质量，还压缩占用空间。如果第一次压缩的压缩质量比例跟第二次压缩的比例的差值，比第一次压缩的压缩占用空间比例跟第二次的压缩占用空间比例的差值小，那么C图片就会比B图片大。因为虽然C图片质量比B图片质量差点，但是B图片的空间压缩得比C图片大比较多。



-END-
欢迎到我的博客交流：https://zackzheng.info