---
title: 为Octopress博客添加SEO
date: 2015-06-07 16:00:00
tags: 
     - Octopress
     - SEO
categories: blog
keywords: Octopress
description: 为Octopress博客添加SEO
---

建好博客，当然希望更多的人看到和交流，做好SEO是必要的。

- ##### 将网站地址提交给各大搜索引擎

为了让自己搭建的博客更容易被搜索引擎搜到，最好将网站地址提交给各大搜索引擎，下面有两个连接搜集了各个搜索引擎的网站提交入口：

```
http://urlc.cn/tool/addurl.html
http://tool.lusongsong.com/addurl.html
```

- ##### 文章头添加更多信息

我们发现创建一篇新文章的时候，生成的makedown文件包含以下内容：

```
layout: post
title: "为Octopress博客添加SEO"
date: 2015-06-07 08:23
comments: true
categories: Octopress
```

实际上我们还可以为其添加以下几项，以本文举例：

```
keywords: octopress blog github pages SEO analytics
description: 为Octopress添加SEO
```

可以修改Rakefile文件，让创建文章时自动添加,找到以下代码并修改如下：

```
puts "Creating new post: #{filename}"
open(filename, 'w') do |post|
post.puts "---"
post.puts "layout: post"
post.puts "title: \"#{title.gsub(/&/,'&')}\""
post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
post.puts "comments: true"
post.puts "categories: "
post.puts "keywords: "
post.puts "description: "
post.puts "---"
```

还有这里：

```
puts "Creating new page: #{file}"
open(file, 'w') do |page|
page.puts "---"
page.puts "layout: page"
page.puts "title: \"#{title}\""
page.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
page.puts "comments: true"
page.puts "keywords: "
page.puts "description: "
page.puts "sharing: true"
page.puts "footer: true"
page.puts "---"
```

- ##### Google Analytics

Google Analytics 是Octopress自带的统计工具，使用方式也非常简单，只需要到 Google Analytics 申请一个 app id ，填写到 _config.yml 文件中的 google_analytics_tracking_id 后面即可。

- ##### 百度统计

百度站长工具和百度统计，注册后将代码添加到source/_includes/custom/footer.html中。



-END-
欢迎到我的博客交流：https://zackzheng.info