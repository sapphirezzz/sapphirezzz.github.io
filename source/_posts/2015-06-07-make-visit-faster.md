---
title: 加速Octopress博客的访问
date: 2015-06-07 16:00:00
tags: 
     - Octopress
categories: blog
keywords: blog Octopress
description: 加速Octopress博客的访问
---

因为“墙”的关系，所以Octopress建立以后会发现访问速度奇慢无比。网上找到几个加快访问的方法。

### 替换Google JS公共库

Octopress默认使用的是Google的JS公共库地址，加载的过程无比缓慢。因此我们要把它改为“百度的JS公共库”，需要修改 /source/_includes/head.html 文件：

```
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
```

改为

```
<script src="//libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
```

可在此查到百度最新的jquery版本的地址：

> <http://developer.baidu.com/wiki/index.php?title=docs/cplat/libs#jQuery>

### 去掉Twitter

把在根目录下的 _config.yml 文件中Twitter内容给注释掉。

```
# Twitter
#twitter_user:
#twitter_tweet_button: true
```

把 \source\_includes\after_footer.html 文件中的Twitter内容给注释掉：

```
{% raw %}<!--{% include twitter_sharing.html %}-->{% endraw %}
```

### 删除Google font

把在 \source\_includes\custom\head.html 中的Google font样式给删除：

```
<link href="//fonts.googleapis.com/css?family=PT+Serif:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">
<link href="//fonts.googleapis.com/css?family=PT+Sans:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">
```



-END-
欢迎到我的博客交流：https://zackzheng.info