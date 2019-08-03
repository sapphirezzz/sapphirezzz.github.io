---
title: 为Octopress博客添加评论功能
date: 2015-06-01 16:00:00
tags: 
     - Octopress
categories: blog
keywords: blog Octopress
description: 为Octopress博客添加评论功能
---

Octopress默认自带了DISQUS，但是对于国内不是很好用。国内比较流行的是多说评论系统。

### 添加多说评论组件

1、去多说 [http://dev.duoshuo.com](http://dev.duoshuo.com/) 注册，设置好站点和对应的多说域名

2、在 _config.yml 中添加

```
# duoshuo comments
duoshuo_comments: true
duoshuo_short_name: “你的多说名称”
```

3、在 ./source/_layouts/post.html 中的 disqus 代码下方添加多说评论模块：

```
{% raw %}
{% if site.duoshuo_short_name and site.duoshuo_comments == true and page.comments == true %}
  	<section>
    	<h1>Comments</h1>
	<div id="comments" aria-live="polite">{% include post/duoshuo_comment.html %}</div>
      	</section>
{% endif %}
{% endraw %}
```

4、创建 ./source/_includes/post/duoshuo_comment.html 文件，内容如下：

```
<!-- Duoshuo Comment BEGIN -->
<div class="ds-thread" data-title="Octopress博客的个性化配置"></div>
<script type="text/javascript">
	var duoshuoQuery = {short_name:"你的多说名称"};
    (function() {
        var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = 'http://static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
</script>
<!-- Duoshuo Comment END -->
```

5、随后，再修改 `_includes/article.html` 文件，在

```
{% raw %}
{% if site.disqus_short_name and page.comments != false and post.comments != false and site.disqus_show_comment_count == true %}
 | <a href="{% if index %}{{ root_url }}{{ post.url }}{% endif %}#disqus_thread">Comments</a>
{% endif %}
{% endraw %}
```

下方添加如下代码：

```
{% raw %}
{% if site.duoshuo_short_name and page.comments != false and post.comments != false and site.duoshuo_comments == true %}
 | <a href="{% if index %}{{ root_url }}{{ post.url }}{% endif %}#comments">Comments</a>
{% endif %}
{% endraw %}
```

其实以上代码只是修改自多说上的而已： <http://dev.duoshuo.com/docs/4ff28d95552860f21f000010>

还有一些多说参数可以配置。



-END-
欢迎到我的博客交流：https://zackzheng.info