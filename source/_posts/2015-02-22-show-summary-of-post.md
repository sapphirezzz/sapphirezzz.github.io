---
title: 让Octopress博文列表显示摘要
date: 2015-02-22 16:00:00
tags: 
     - Octopress
categories: blog
keywords: blog Octopress
description: 让Octopress博文列表显示摘要
---

默认情况下，博客首页文章列表会展示文章全文，可以配置成只展现一部分。

首先在每篇文章你想展示的缩略部分最后增加标记：

```
<!-- more -->
```

修改_config.yml中的对应设置项为：

```
excerpt_link: "阅读更多 →"
```

然后文件保存为UTF-8字符编码格式。

如果希望默认不显示摘要和内容，并且在新建的时候就配置好，可以修改 Rakefile 文件。

```
puts "Creating new post: #{filename}"
	open(filename, 'w') do |post|
	post.puts "---"
    ......
```

在上面一段的最后面加上：

```
post.puts "<!-- more -->"
```



-END-
欢迎到我的博客交流：https://zackzheng.info