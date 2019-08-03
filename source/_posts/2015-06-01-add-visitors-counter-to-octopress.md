---
title: 为Octopress博客添加访客统计功能
date: 2015-06-01 16:00:00
tags: 
     - Octopress
categories: blog
keywords: blog Octopress
description: 为Octopress博客添加访客统计功能
---

有两种访客统计系统：Flag Counter和多说 [http://dev.duoshuo.com](http://dev.duoshuo.com/) 的访客组件

Flag Counter是显示各个国家访客数，多说访客组件是显示访客头像。

### Flag Counter

1、先去Flag Counter [http://www.flagcounter.com](http://www.flagcounter.com/) 获取代码。

2、拿到代码后添加 .\source\_includes\custom\asides\flag_counter.html 文件：

flag_counter.html

```
<section>
<h1>访客统计</h1>
<br/>
<a href="http://s07.flagcounter.com/more/2SH"><img src="http://s07.flagcounter.com/count/2SH/bg_FFFFFF/txt_000000/border_CCCCCC/columns_1/maxflags_20/viewers_0/labels_0/pageviews_1/flags_0/" alt="Flag Counter" border="0"></a>
</section>
```

代码包含中文则需要另存为UTF-8格式。

3、将页面添加到侧边栏，在 ./_config.yml 配置文件中添加配置：

如果有default_asides，则在后面中括号中添加

```
， custom/asides/flag_counter.html
```

没有则在文件中添加

```
default_asides: [custom/asides/flag_counter.html]
```

4、最后添加控制开关，在 ./_config.yml 配置文件中添加下面一行配置：

```
# Flag Counter
flag_counter: true
```

### 多说访客系统

1、去多说 [http://dev.duoshuo.com](http://dev.duoshuo.com/) 注册，设置好站点和对应的多说域名

2、在 <http://dev.duoshuo.com/docs/4ff28d6f552860f21f00000c> 获取代码，在前后添加section标签。可以在section中添加任何代码。

多说参数添加在class后面，如下：

```
<section>
<ul class="ds-recent-visitors" data-num-items="10"></ul>
<!--多说js加载开始，一个页面只需要加载一次 -->
<script type="text/javascript">
	var duoshuoQuery = {short_name:"您的多说域名"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = 'http://static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
</script>
<!--多说js加载结束，一个页面只需要加载一次 -->
</section>
```

3、同理，继续上面Flag Counter的第2、3、4步即可。



-END-
欢迎到我的博客交流：https://zackzheng.info