---
layout: post
title: "Octopress Seo"
date: 2012-08-29 21:50
comments: true
categories: [Octopress]
---
用 Google 大婶搜索了下 Octopress 可以定制的内容，发现可以定制的东西真的还蛮多的，也基本都是平常能看得的几个功能，比如下面几个：

- [给octopress博客添加豆瓣侧边栏](http://tinyxd.me/blog/2012/07/09/add-douban-aside/) 
- [添加新浪微博侧边栏](http://caok.github.com/blog/2012/06/24/install-octopress-to-write-blog/)
<!--more-->
- [给octopress加上标签云](http://log4d.com/2012/05/tag-cloud/)
- [添加about，友情链接](http://shanewfx.github.com/blog/2012/08/13/improve-blog-theme/)

不如我一向遵循 `less is more, more is less` 的原则，不想把 blog 弄的太臃肿，而且这些功能对我也基本用处不大，没有微博上碎嘴的习惯，豆瓣上也只去电影区看影评，tag云感觉和category重复，友链暂时也没有，等等。先存个链接，说不定什么时候就想加上。。

目前先保持简洁的风格，稍微做下 seo 优化，过程参考 [这篇](http://www.yatishmehta.in/seo-for-octopress)

##### 添加 meta data description

文章生产时默认会生成 layout, title, date, comments, categories 这几项，如果多添加2个选项 'keywords', 'description'，会在 generate 文章的时候自动生成相应的 meta 标签

如果文章 header 写成这样

```html
layout: post
title: "SEO for Octopress"
date: 2012-04-22 09:55
comments: true
categories: [seo,octopress]
keywords: seo,octopress,heroku
description: How to optimize Octopress for SEO,Heroku
```

生成的post head标签中会生成

```
<title>SEO for Octopress </title>
<meta name="author" content="Yatish Mehta">
<meta name="description" content="How to optimize Octopress for SEO">
<meta name="keywords" content="seo,octopress">
```

如果不想每次都得手动添加，可以修改 Rakefile - new_post task任务

```ruby
open(filename, 'w') do |post|
  post.puts "---"
  post.puts "layout: post"
  post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
  post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
  post.puts "comments: true"
  post.puts "categories: "
  post.puts "keywords: "
  post.puts "description: "
  post.puts "---"
end
```

##### 主页的 meta data

上面的方法只能给每个 post 生成对应的 meta标签，如果要给主页添加 meta 标签，修改 source/_includes/head.html，添加以下内容

```ruby
<meta name="author" content="{{ site.author }}">
{% capture description %}{% if page.description %}{{ page.description }}{% elsif site.description %}{{ site.description }}{%else%}{{ content | raw_content }}{% endif %}{% endcapture %}
<meta name="description" content="{{ description | strip_html | condense_spaces | truncate:150 }}">
{% if page.keywords %}<meta name="keywords" content="{{ page.keywords }}">{%else%}<meta name="keywords" content="{{ site.keywords }}">{% endif %}
```

这段内容其实已经被引入了 Octopress 源码里面了，所以只需要在 _config.yml 文件中填充全局的 keywords 和 description, 比如这样

```
description: Tips, tricks, and hacks on Ruby on Rails development.
keywords: ruby,ruby on rails,salesforce,gem,web development,Ajax,SEO,scraping
```

这样 /blog 主页就会自动生成对应的 meta

##### 指向单独的域名

有时候你的域名可能同时对带 www 前缀和不带前缀同时有效，这种情况有可能会影响网站的 page rank，你需要把其中的一个作为主域名，重定向另一个指向这个域名

以上只是一些简单的 seo 技巧，本着有总比没有强点的原则稍加修改就OK了