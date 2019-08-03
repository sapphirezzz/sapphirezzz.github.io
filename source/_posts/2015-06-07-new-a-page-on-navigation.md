---
title: 在Octopress博客的导航栏添加页面
date: 2015-06-07 16:00:00
tags: 
     - Octopress
categories: blog
keywords: blog Octopress
description: 在Octopress博客的导航栏添加页面
---

在octopress中，已经有两个默认page，即blog、archives，我们可以参考它来完成自己的页面。

执行一下语句：

```
rake new_page["about"]
```

此时在目录下会生成文件夹 about ，编辑文件夹内的文件 index.markdown，即你新页面的内容。

编辑 source/_includes/custom/navigation.html

```
<ul class="main-navigation">
	<li><a href="/">Blog</a></li>
	<li><a href="/blog/archives">Archives</a></li>
	<li><a href="/jsccp">JavaScript内核系列</a></li>
</ul>
```

添加了一个新行，指向新创建的目录。

执行：

```
rake gen_deploy
```



-END-
欢迎到我的博客交流：https://zackzheng.info