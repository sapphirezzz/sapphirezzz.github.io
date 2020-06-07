---
title: 手把手教你Promise小技巧——小程序接口数限制下的并行再顺序执行
date: 2020-06-07 18:00:00
tags: 
     - Taro
     - Promise
categories: Tech
description: 通过小程序上传接口并发数限制下进行上传的场景，分享 Promise 使用 reduce 实现顺序执行。

---

# 一、前言

`Promise`是一种异步编程的解决方案，在很多语言都有它的实现，笔者一直喜欢使用它去解决各种问题，简洁高效。

`ECMAscript 6`原生提供了`Promise`对象。本文通过小程序上传接口并发数限制下进行上传的场景，实现图片并行再串行上传，分享`Promise`顺序执行的一种实现。

# 二、实现讲解

在[微信官方文档·小程序-网络使用说明](https://developers.weixin.qq.com/miniprogram/dev/framework/ability/network.html)中，对`wx.request`、`wx.uploadFile`、`wx.downloadFile`的最大并发限制是 **10** 个。因此当遇到一个页面需要上传超过10个文件时，无法使用`Promise`并发调用上传接口。

比如页面有两个产品元素，一个元素需要上传5张照片，一个元素需要上传8张照片。此时我们操作后得到两个数组，一个包含5个上传的`Promise`，一个包含8个上传的`Promise`。我们可以数组内并行发起，数组间则顺序执行。

这样得到的逻辑就是，先并行上传第一个元素的5张照片，上传完再并行上传第二个元素的8张照片。（当然也可以将所有的上传按10个分批再顺序执行）

接下里我们一步步，从浅到深，分享下处理的思路。
## 1. 顺序执行两个Promise
在两个异步操作的情况下，使用`then`将两个`Promise`串起来：
```typescript
a.then( _ => b );
```



## 2. 顺序执行多个Promise

在异步操作个数不确定的情况下，可以使用`reduce`将它们串起来：
```typescript
// promiseArray 是数组，数组内对象是 Promise 类型
promiseArray.reduce( (p, f) => p.then(_ => f) );
```

使用`reduce`便是实现顺序执行的基础，理解清楚这行代码发生了什么，处理其他问题便可迎刃而解。

此处其实是第1点的高阶函数写法，同样是执行完前一个`Promise`之后，执行后一个`Promise`，然后将结果作为`Result`进行下一次循环。因此需要注意到，最终结果只包含了最后一个`Promise`的执行结果，没有其他`promise`的执行结果。



## 3. 顺序执行并获取所有结果

显然，我们是需要得到所有顺序执行的结果的，因此需要进行一定的处理。

我们可以对`reduce`内每一步的结果进行处理，组合成一个数组，此时需要用到`Promise.all`。

```typescript
// promiseArray 是数组，数组内对象是 Promise 类型
promiseArray.reduce(
  (p, f) => p.then(
    res => Promise
      .all([Promise.resolve(res), f])
      .then( res => Promise.resolve( res[0].concat([res[1]]) )
  )
), Promise.resolve([]));
// 注意此处需要提供初始值，否则会出现不符合预期的结果
```

这是最终的处理，下面一步步讲解下。

上面第一个`res`是数组类型，存放了前面每个`Promise`执行成功后返回的结果。

```typescript
Promise.all([Promise.resolve(res), f]);
```

此代码是用第一个`res`构造一个`Promise`，和`f`一起并行执行，最终会得到第二个`res`，它是一个数组，数组第一个对象是第一个`res`，第二个对象是`f`成功返回的数据。

```typescript
Promise.resolve( res[0].concat([res[1]]);
```

此处将第二个`res`的第一个对象(数组类型)和第二个对象合并成一个新数组，并构造成一个`Promise`。

因此，最终会得到一个数组，该数组包含所有需要顺序执行的`Promise`的结果。
举例如下：

```typescript
let promise1 = Promise.resolve('1');
let promise2 = Promise.resolve('2');
let promise3 = Promise.resolve('3');
[promise1, promise2, promise3].reduce((p, f) => p.then(res => Promise
      .all([Promise.resolve(res), f])
      .then( res => Promise.resolve(res[0].concat([res[1]])) ) 
), Promise.resolve([])).then( res => {
	console.log(res);
});
```

运行结果：

```
> Array ["1", "2", "3"]
```



## 4. 先并行再顺序执行得到所有结果

理解了第3点，那么我们只需要将每个顺序执行的`Promise`，替换成多个并行执行的`Promise`。

举例如下：

```typescript
let promise1 = Promise.all([Promise.resolve('11'), Promise.resolve('12'), Promise.resolve('13')]);
let promise2 = Promise.all([Promise.resolve('21'), Promise.resolve('22'), Promise.resolve('23')]);
let promise3 = Promise.all([Promise.resolve('31'), Promise.resolve('32'), Promise.resolve('33')]);
[promise1, promise2, promise3].reduce(
  (p, f) => p.then(
    res => Promise.all([Promise.resolve(res), f]).then( res => Promise.resolve(res[0].concat([res[1]])) ) 
  )
  , Promise.resolve([])
)
.then( res => {
    console.log(res);
});
```

运行结果：

```typescript
> Array [Array ["11", "12", "13"], Array ["21", "22", "23"], Array ["31", "32", "33"]]
```

如果想要全部放一组：
```typescript
let promise1 = Promise.all([Promise.resolve('11'), Promise.resolve('12'), Promise.resolve('13')]);
let promise2 = Promise.all([Promise.resolve('21'), Promise.resolve('22'), Promise.resolve('23')]);
let promise3 = Promise.all([Promise.resolve('31'), Promise.resolve('32'), Promise.resolve('33')]);
[promise1, promise2, promise3].reduce(
  (p, f) => p.then(
    res => Promise.all([Promise.resolve(res), f]).then( res => Promise.resolve(res[0].concat(res[1])) ) 
  )
  , Promise.resolve([])
)
.then( res => {
    console.log(res);
})
```
运行结果：
```
> Array ["11", "12", "13", "21", "22", "23", "31", "32", "33"]
```
区别只在`concat `的地方。

# 三、总结

其实复杂的`Promise`难写，也不难写。结合`reduce`，需要先考虑最终想要什么类型的数据，再对其进行处理，在强类型的语言中如ts、Swift，需要写明初始值的类型，不然会导致出错，耗费较长时间。

 

-END-
欢迎到我的博客交流：[https://zackzheng.info](https://links.jianshu.com/go?to=https%3A%2F%2Fzackzheng.info)