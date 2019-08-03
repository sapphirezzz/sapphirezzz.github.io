---
title: 将博客从Octopress迁移往Hexo
date: 2016-03-06 16:00:00
tags: 
     - blog
     - Hexo
     - Octopress
categories: blog
keywords: Octopress Hexo
description: 将博客从Octopress迁移往Hexo
---

# 目录
- 前言
- 迁移到Hexo
- 配置Hexo
- 常用命令
- 总结

# 前言
之前一直使用Octopress，折腾过好多次，出问题不少，而且生成文章速度越来越慢，需要自己添加各种插件，还弄过支持TOC一直没成功。最近再看了下迁移到hexo的，好像蛮方便的，而且是基于Node的，这让我特别想试下，没想到结果出乎意料的好用！

# 迁移到Hexo

**安装nodejs**：[下载链接](https://nodejs.org/en/download/)

**安装Hexo**：

```
$ npm install -g hexo-cli
```

**初始化目录**：

```
$ hexo init <blog>
$ cd <blog>
$ npm install
```

完成后便生成了以下目录结构：

> .
> ├── _config.yml
> ├── package.json
> ├── scaffolds
> ├── source
> | ├── _drafts
> | └── _posts
> └── themes

- _config.yml：配置文件
- source：文章目录
- themes：主题目录

**修改信息**：

修改_config.yml内容：

```
title: Just My Blog
subtitle: A Programmer Who Wants To Be A Poet .----尘世间一件迷途小码农
description: Zack Zheng's Blog
author: Zack Zheng
```

**迁移文章**：

将以前Octopress的source/_post目录下的文章拷贝到Hexo的同名目录下即可。

**安装hexo-deployer-git**：

```
$ npm install hexo-deployer-git --save
```

**修改_config.yml最后面几行**：

```
deploy:
  type: git
  repository: https://github.com/<name>/<name>.github.io.git
  branch: master
```

**配置域名**：

将之前的CNAME文件放到source目录下即可。

**上传到Github**：

将原来在Github上的Repository删掉重建，格式依旧是<*name*>.github.io。

执行下面命令编译成静态文件并提交：

```
$ hexo generate
$ hexo deploy
```

# 配置Hexo

之前Octopress还弄过侧边栏、多说评论、摘要、访客记录、回到顶部、谷歌统计等，现在没有怪可惜的，但发现Hexo的主题Jacman居然已经帮忙支持了这些功能，还有TOC，这实在是令人欣喜！

**安装主题**：

```
$ cd <blog>
$ git clone https://github.com/A-limon/pacman.git themes/pacman
```

**使用主题**：

修改_config.yml文件如下：

```
theme: jacman
stylus:
  compress: true
```

compress值为true，会自动压缩 CSS 文件。

**更新主题**：

```
$ cd themes/jacman
$ git pull origin master
```

**修改主题**：

- 修改主题配置

根据自己的需要修改themes/jacman/_config.yml文件。

去掉Widgets下不需要的组件，如以下修改：

```
#### Widgets
widgets: 
- github-card
- category
- tag
## - links
## - douban
- rss
## - weibo
- duoshuo_visitor
- flag_counter
```

duoshuo_visitor和flag_counter不是自带的，是后面需要另外加的。

修改底部footer，如以下修改：

```
#### Author information
author:
  intro_line1:  "Hello ,I'm Zack Zheng."    ## your introduction on the bottom of the page
  intro_line2:  "This is my blog."  ## the 2nd line
  email: zhengzuanzhe@gmail.com     ## e.g. imjark@gmail.com
```

在duoshuo_shortname: 后面空格输入你的多说用户名即可支持评论。

后边还支持对谷歌统计、百度统计的配置。

- 替换默认图片资源

替换themes\jacman\source\img下的几个图片

- 增加多说访问统计和flag_counter

1. 在_config.yml的widgets下面加入插件名
2. 在themes/jacman/layout/_widget目录下新建文件duoshuo_visitor.ejs、flag_counter.ejs。

duoshuo_visitor.ejs(short_name后面是多说用户名)：

```
<section>
<br/>
<h1>最近访客</h1>
<br/>
<ul class="ds-recent-visitors" data-num-items="20"></ul>
<script type="text/javascript">
  var duoshuoQuery = {short_name:"zackzheng"};
  (function() {
      var ds = document.createElement('script');
      ds.type = 'text/javascript';ds.async = true;
      ds.src = 'http://static.duoshuo.com/embed.js';
      ds.charset = 'UTF-8';
      (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(ds);
  })();
</script>
<br/>
</section>
```

flag_counter.ejs(a href后面替换成自己在[Flag Counter](http://www.flagcounter.com/)获取的链接)：

```
<section>
<h1>访客统计</h1>
<br/>
<a href="http://s07.flagcounter.com/more/2SH"><img src="http://s07.flagcounter.com/count/2SH/bg_FFFFFF/txt_000000/border_CCCCCC/columns_1/maxflags_20/viewers_3/labels_0/pageviews_1/flags_0/" alt="Flag Counter" border="0"></a>
</section>
```

# 常用命令

> hexo new ““ #新建文章
> hexo new page ““ #新建页面
> hexo generate #生成静态页面至public目录
> hexo server #开启预览访问端口（默认端口4000，’ctrl + c’关闭server）
> hexo deploy #将.deploy目录部署到GitHub
> hexo help # 查看帮助
> hexo version #查看Hexo的版本
>
> hexo deploy -g #生成加部署
> hexo server -g #生成加预览

命令简写：

> hexo n = hexo new
> hexo g = hexo generate
> hexo s = hexo server
> hexo d = hexo deploy

# 总结

总体来说，切换到Hexo和Jacman主题后，可以很方便设置多说评论、谷歌统计、百度统计，并自带分享、侧边栏，支持TOC，不用接触ruby等代码，不易出问题，生成速度快。



-END-
欢迎到我的博客交流：https://zackzheng.info