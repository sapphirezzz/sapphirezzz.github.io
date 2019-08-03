---
title: 为基于Octopress的博客添加“回到顶部”的功能
date: 2015-02-22 16:00:00
tags: 
     - Octopress
categories: blog
keywords: blog Octopress
description: 为基于Octopress的博客添加“回到顶部”的功能
---



> <http://www.scrolltotop.com/>

这个网站提供了数十种回到顶部的样式，可以根据自己的需要，添加所中意的widget。

### 如何添加

在source\_includes\custom目录下新建scroll_to_top.html，存放widget代码。

```
<script type="text/javascript" src="http://arrow.scrolltotop.com/arrow64.js"></script>
<noscript>Not seeing a <a href="http://www.scrolltotop.com/">Scroll to Top Button</a>? Go to our FAQ page for more info.</noscript>
```

默认Octopress引入了jquery.min.js，所以没有必要再次引入。

回到顶部功能，不仅仅要在文章页生效，同时也需要对类似归档页面有效才完美。于是需要找一个公用的位置。这个位置就是source\_layouts\default.html。

修改default.html文件,在body体末尾添加:

```
{% raw %}{% include custom/scroll_to_top.html %}{% endraw %}
```

到这里，“回到顶部”的功能完成。

### 相关链接

Octopress添加回到顶部功能

> <http://www.tuicool.com/articles/qu6ZfiV>



-END-
欢迎到我的博客交流：https://zackzheng.info