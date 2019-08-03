---
title: Github Pages,Ruby,Markdown,Jekyll and Octopress
date: 2015-06-07 16:00:00
tags: 
     - blog
categories: Tech
keywords: blog Octopress Ruby
description: Github Pages,Ruby,Markdown,Jekyll and Octopress
---

# 目录
- 前言
- 各种框架和技术
- 小总结

# 前言

搭建了这个基于Octopress和Github Pages的博客已经几个月了，但是其实对这里面的一些技术和原理还不了解，导致有时候在配置的时候会出点问题。所以觉得有必要简单梳理下。

# 各种框架和技术

## Github Pages

\#### 

首先需要说到Github，它是一个具有版本管理功能的代码仓库，每个项目都有一个主页，列出项目的源文件。

*但是对于一个新手来说，看到一大堆源码，只会让人头晕脑涨，不知何处入手。他希望看到的是，一个简明易懂的网页，说明每一步应该怎么做。因此，github就设计了Pages功能，允许用户自定义项目首页，用来替代默认的源码列表。所以，github Pages可以被认为是用户编写的、托管在github上的静态网页。*

## Jekyll

看它的官网 <http://jekyll.bootcss.com/> 介绍：

*将纯文本转化为静态网站和博客，只用 Markdown (或 Textile)、Liquid、HTML & CSS 就可以构建可部署的静态网站。*

所以它是一个静态网页文件生成器，然后支持了Github Pages。

Jekyll是基于Ruby的，所以需要安装Ruby和Ruby Devkit。

## Octopress

首先还是看它自己的介绍 <https://github.com/imathis/octopress> ：

*Octopress is an obsessively designed framework for Jekyll blogging. It’s easy to configure and easy to deploy.* 

**所以它是基于Jekyll的，并且内置了不少博客框架，让你搭建博客更方便。**

在以下文件可以看到它的组成：

<https://github.com/imathis/octopress/blob/master/Gemfile>

**网上看到一个区别**：

上传发布细微区别，jekyll 可以直接用git上传 .md or .html 文件不需要编译，Otcopress是makefile将文件转化.html发布。所以这也是一个缺点，当文章越多，编译发布时间就越长了吧。

另外，Octopress 在 Jekyll 0.x 时代是个非常好的 Jekyll-blog 解决方案，有不错的模版，丰富的扩展功能，缺点就是麻烦，需要在本地生成页面。但是在过去一年，Jekyll大量的新功能新特性加入让 Octopress 显得不那么必要。Octopress 3 也改变策略，不再那么复杂，只是对 Jekyll 操作进行二次封装，方便使用。

## Ruby

一种脚本语言。Octopress是基于Ruby的，所以我们不时需要修改这种语言的代码。

## Markdown

\#### 

Markdown是一种可以使用普通文本编辑器编写的标记语言，通过类似HTML的标记语法，它可以使普通文本内容具有一定的格式。

## Rake

**网上的介绍**：

*Rake 是用 Ruby 编写的，并使用 Ruby 作为它的语法，是 Ruby 的 Make ，许多方面都比 Make 要更好用一些。

看这个文件 <https://github.com/imathis/octopress/blob/master/Gemfile> 知道，它包含在Octopress中。

## Rakefile

\#### 

**网上的介绍**：

和 Makefile 不同的是，Rakefile 本身其实就是一段 Ruby 代码，这样的好处有很多，一方面在 Rake 里面就可以很直接地做任何 Ruby 能做的事了，另一方面由于 Ruby 对 DSL 支持良好，所以 Rakefile 通常看起来也并不那么“代码”。*

## Gemfile

Gemfile 文件描述Ruby工程需要依赖的插件bundle。

执行以下两句命令的时候就是安装bundler插件，然后把 Gemfile 文件里面关联的插件下载下来。

```
gem install bundler
bundle install
```

# 小总结

**所以，总结一下这种搭建博客的方案的实现步骤**：

1. 首先我们使用 rake new_post[“XXX”] 生成一个 markdown 文件（rake命令来自rake工具，从目录下的 Rakefile 文件可以看到这条命令做了什么），然后编写这个文件。
2. 使用命令 rake generate ,会将 markdown 文件转换成 html 文件。在这个过程中利用了Octopress的其他插件，配置了如头部、尾部、侧边栏、评论等等。
3. 使用 rake deploy 命令将文件都上传到 Github Pages 上。然后我们访问那个域名就可以看到了。
4. 如果要使用自己的域名，就需要让自己的域名解析指向这个地址，具体可以看我的另一篇文章（<http://zackzheng.info/blog/2015/02/22/start-a-blog-on-github-pages-base-on-octopress/> ）

这里讲的还是比较浅显，还有一些，比如 rake generate 到底做了什么，整个目录下各个文件夹分别是什么作用等等，还没怎么了解，如果了解的话对以后出问题的解决还是有帮助的。

总的来说，网上可以找到很多建博客的方法和它们的优劣对比，这里就不多说那些，个人觉得最终还是要回归到自己建博客的目的：写文章，不然迁来迁去挺折腾的，所以在写和发表时效率高这个最重要了，其他都是次要。



-END-
欢迎到我的博客交流：https://zackzheng.info