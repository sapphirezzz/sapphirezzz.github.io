---
title: 在Octopress博客中避免使用的代码块被解析
date: 2015-06-06 16:00:00
tags: 
     - Octopress
categories: blog
keywords: blog Octopress
description: 在Octopress博客中避免使用的代码块被解析
---
# 前言

在写上一篇集成多说评论的文章的时候，遇到一个代码高亮的问题。

# 问题描述

作为嵌入的一些代码会被解析并转成其真实的值对应的HTML代码形式。

# 解决办法

想避免嵌入的代码块被解析，使用{% raw %}和{% endraw %}来包裹不想被解析的代码块即可。

# 实际例子

下面这段代码块看似可以正常显示，其实是使用上面的方法，如果不包裹会出现奇怪的现象。

```
{% raw %}
{% if site.duoshuo_short_name and site.duoshuo_comments == true and page.comments == true %}
```

```
<section>
```

```
<h1>Comments</h1>
<div id="comments" aria-live="polite">{% include post/duoshuo.html %}</div>
```

```
</section>
```

```
{% endif %}
{% endraw %}
```

# 更棘手的

如果出现了Liquid Exception: Unknown tag ‘endraw’ in _posts这样的问题， 使用&#123;代替{ ,使用&#125;代替}

原文链接:  <http://droidyue.com/blog/2014/08/18/how-to-escape-embeded-ruby-block-in-octopress/>



-END-
欢迎到我的博客交流：https://zackzheng.info