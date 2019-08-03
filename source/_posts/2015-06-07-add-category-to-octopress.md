---
title: 为Octopressbo博客添加侧边栏
date: 2015-06-07 16:00:00
tags: 
     - Octopress
categories: blog
keywords: blog Octopress
description: 为Octopressbo博客添加侧边栏
---

### 添加侧边栏文章分类

1.在 plugins 目录下创建 category_list_tag.rb 文件，内容如下：

```
module Jekyll 
	class CategoryListTag < Liquid::Tag 
		def render(context) 
			html = "" 
			categories = context.registers[:site].categories.keys 
			categories.sort.each do |category| 
			posts_in_category = context.registers[:site].categories[category].size 
			category_dir = context.registers[:site].config['category_dir'] 
			category_url = File.join(category_dir, category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase) 
			html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n" 
			end 
			html 
		end 
	end 
end

Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
```

2.添加 source/_includes/asides/category_list.html 文件，内容如下：

```
{% raw %}
<section>
	<h1>文章分类</h1>
	<ul id="categories">
		{% category_list %}
	</ul>
</section>
{% endraw %}
```

注意另存为UTF-8格式。

3.修改 _config.yml 文件，在 default_asides 项中添加 asides/category_list.html ，值之间以逗号隔开，值的先后顺序代表了侧边栏展现的先后顺序。

```
default_asides: [asides/category_list.html, asides/recent_posts.html, asides/github.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html]
```

在侧边栏还可以添加其他组件，如微博、标签云等，添加方式和上面类似。



-END-
欢迎到我的博客交流：https://zackzheng.info