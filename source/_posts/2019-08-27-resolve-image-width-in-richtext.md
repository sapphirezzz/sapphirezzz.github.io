---
title: 解决Taro下RichText遇到的图片宽度问题
date: 2019-08-27 18:00:00
tags: 
     - Taro
     - RichText
categories: Tech
description: 解决Taro下RichText遇到的图片宽度问题

---

使用`Taro`的`RichText`组件，遇到展示图片的宽度超出屏幕的问题，记录下解决办法。可以根据实际需要，对以下方法进行修改。

- 只有图片标签，并且图片标签没有设置style和宽度/高度

```js
function formatRichText(html){
  let newContent= html.replace(/\<img/gi, '<img style="width:100%;height:auto"');
  return newContent;
}
```

- 包含多种标签

```js
/**
 * 处理富文本里的图片宽度自适应
 * 1.去掉img标签里的style、width、height属性
 * 2.img标签添加style属性：max-width:100%;height:auto
 * 3.修改所有style里的width属性为max-width:100%
 * 4.去掉&lt;br/&gt;标签
 * @param html
 * @returns {void|string|*}
 */
function formatRichText(html){
  let newContent= html.replace(/&lt;img[^&gt;]*&gt;/gi,function(match,capture){
    match = match.replace(/style=&quot;[^&quot;]+&quot;/gi, '').replace(/style='[^']+'/gi, '');
    match = match.replace(/width=&quot;[^&quot;]+&quot;/gi, '').replace(/width='[^']+'/gi, '');
    match = match.replace(/height=&quot;[^&quot;]+&quot;/gi, '').replace(/height='[^']+'/gi, '');
    return match;
  });
  newContent = newContent.replace(/style=&quot;[^&quot;]+&quot;/gi,function(match,capture){
    match = match.replace(/width:[^;]+;/gi, 'max-width:100%;').replace(/width:[^;]+;/gi, 'max-width:100%;');
    return match;
  });
  newContent = newContent.replace(/&lt;br[^&gt;]*\/&gt;/gi, '');
  newContent = newContent.replace(/\&lt;img/gi, '&lt;img style=&quot;max-width:100%;height:auto;display:block;margin-top:0;margin-bottom:0;&quot;');
  return newContent;
}
```



-END-
欢迎到我的博客交流：[https://zackzheng.info](https://zackzheng.info)