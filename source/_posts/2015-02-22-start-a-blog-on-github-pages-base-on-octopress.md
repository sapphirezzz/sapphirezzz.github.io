---
title: Windows下基于Github Pages创建Octopress博客
date: 2015-02-22 16:00:00
tags: 
     - Octopress
categories: blog
keywords: blog Octopress
description: Windows下基于Github Pages创建Octopress博客
---

# 安装和配置必要环境

## Git

> <http://git-scm.com/downloads>

1. 安装后在github上新建仓库，命名格式为*USERNAME*.github.io
2. 配置SSH密钥：<https://help.github.com/articles/generating-ssh-keys/>

## Ruby

> <http://rubyinstaller.org/downloads/>

（Windows操作系统的还需额外安装**RubyDevKit**，下载地址同上）

RubyDevKit下载后解压，进入解压后的文件夹，执行

```
ruby dk.rb init
```

注：两个安装路径中都不能有空格。

如果安装有多个ruby版本，请修改DevKit安装目录下的config.yml。

```
ruby dk.rb install
```

## Octopress

```
git clone git://github.com/imathis/octopress.git YOUR_BLOG_PATH
cd YOUR_BLOG_PATH
gem install bundler
bundle install
rake install（安装Octopress默认主题）
rake setup_github_pages(根据提示输入仓库地址)
rake generate（生成静态页面，把source的内容放到public）
rake preview（可在http://localhost:4000预览）
```

如果网速较慢可以切换到淘宝源：

```
gem sources -a https://ruby.taobao.org/
```

在执行`bundle install`的时候使用的源是目录下的Gemfile文件中的第一行，可以修改后再执行：

```
source 'https://ruby.taobao.org/'
```

## 配置博客

修改 _config.yml 文件，这个文件包含 Main Configs、Jekyll & Plugins 和 3rd Party Settings 三个部分。在这里，我们只需要修改 Main Configs 中的 title 、 subtitle 和 author 。注意如果包含中文则保存为UTF-8。

```
title: XXX
subtitle: XXX
author: XXX
```

# 域名购买和配置

- 在godaddy网站购买域名：[https://www.godaddy.com](https://www.godaddy.com/)

- 登录godaddy，新建一个域名，Domain Name为*YOUR_DOMAIN_NAME*，将Nameservers修改为**F1G1NS1.DNSPOD.NET**和**F1G1NS2.DNSPOD.NET**。

- 在dnspod注册一个帐号：<https://www.dnspod.cn/>

- 登录dnspod，添加域名，然后添加两条记录如下：

| 主机记录 | 记录类型 | 记录值               |
| -------- | -------- | -------------------- |
| @        | A        | 199.27.76.133        |
| www      | CNAME    | *USERNAME.github.io* |

（A记录的记录值可能会变，参考<https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/> ）

- 在*YOUR_BLOG_PATH\source*目录下创建文本文件，命名为CNAME，输入*YOUR_DOMAIN_NAME*，保存退出。

- 在*YOUR_BLOG_PATH*目录下命令行执行以下命令：

```
rake generate
rake deploy
```

- 完成后打开链接即可看到博客内容。

# 编写博客

安装markdown编辑器，本文使用的是Typora

> <http://typora.io/>

在*YOUR_BLOG_PATH\source*目录下执行

```
rake new_post["title"]
```

此时会在*YOUR_BLOG_PATH\source_posts*目录下生成一个.markdown文件，文件名格式为**YYYY-MM-DD-word-word.markdown**，用markdown编辑器编辑该文件，编写后执行以下命令：

```
rake generate
rake deploy
```

以上语句可以合并为

```
rake gen_deploy
```

要生成页面可以用rake new_page：

```
rake new_page["about"]
```

页面即相当于点击导航栏的Blog和Archives看到的页面，Blog和Archives其实就是两个页面。

# 相关说明

*USERNAME*：在github上注册的用户名

*YOUR_BLOG_PATH*：本地存放的博客路径

*YOUR_DOMAIN_NAME*：购买的域名名称



-END-
欢迎到我的博客交流：https://zackzheng.info