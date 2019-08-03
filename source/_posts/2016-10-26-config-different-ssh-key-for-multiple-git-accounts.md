---
title: 为多个不同的git帐号配置ssh key
date: 2016-10-26 22:01:01
tags: 
     - ssh
     - git
categories: Tech
keywords: ssh key git GitHub
description: 简单介绍了如何通过配置～/.ssh/config文件，实现操作多个不同的git帐号，使用不同的ssh key去拉取、提交代码。
---

# 前言

开发中偶尔会遇到本地有多个git帐号，导致提交代码时因权限问题而失败的情况。处理起来不麻烦，就是比较繁琐容易忘记，所以记录下来备忘。



# 生成ssh key

如果已经生成多个不同的key，请跳过这一节。

打开终端运行以下命令：

```shell
ssh-keygen -t rsa -b 4096 -C "[your email]"
```

> [your email]替换成你的邮箱

一直按回车，最后一步命名生成的`key`名称，此时就会在`~/.ssh/`目录下生成对应的文件 `id_rsa_xxx`、`id_rsa_xxx.pub`。执行多次命令生成多个`ssh key`。

执行以下命令复制公钥：

```shell
pbcopy < ~/.ssh/id_rsa_xxx.pub
```

> id_rsa_xxx 是你重命名的名称

执行完就可以到对应的git仓库平台粘贴你的公钥，比如github上`项目->setting->Deploy keys->add deploy key`。

另外，为了避免每次连接时可能会要求输入私钥的对称加密密匙，可以把`key`加入到`authentication agen`t中。需要运行以下命令：

```shell
ssh-add ~/.ssh/id_rsa_xxx.pub
```

 输入你的私钥密码，就可以把私钥加入到ssh-agent中



# 配置config文件

打开终端输入命令：

```shell
vi ~/.ssh/config
```

对内容修改如下（没有则新建）：

```
Host github.com
 HostName github.com
 User git
 IdentityFile ~/.ssh/id_rsa_1

Host code.aliyun.com
 HostName code.aliyun.com
 User git
 IdentityFile ~/.ssh/id_rsa_2
```

> Host 可以随意命名，用于区分不同主机；
>
> HostName 是主机名，也就是仓库 ssh 链接中的主机地址，github 是 github.com
>
> User 是指定登录用户名，github 平台是 git；
>
> IdentifyFile 是指定的私钥地址



# 测试

终端运行以下命令：

```shell
ssh -T git@[your host]
```

> [your host] 分别替换成配置文件中的 Host



# 配置remote url

有几种方式配置远程remote url：

- 修改`.git/config`文件

打开本地仓库的`.git/config`文件，更改`[remote "origin"]`项中的`url`的值为上面对应配置的`Host`值。

- 终端修改

打开终端，进入本地仓库文件夹，运行一下命令：

```shell
git remote add origin [your User]@[your host]:[your git name]/blog.git
```

> [your User] 和 [your host] 分别修改为配置文件中对应配置的值。[your git name] 是你帐号在 git 平台的名称，和仓库链接中的 git 用户名一样。



-END-
欢迎到我的博客交流：https://zackzheng.info