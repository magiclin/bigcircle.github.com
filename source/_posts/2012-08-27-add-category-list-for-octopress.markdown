---
layout: post
title: "给Octopress添加分类列表"
date: 2012-08-27 11:24
comments: true
categories: [octopress]
---
Octopress可定制的功能很多，可以在侧边栏添加分类列表，最近评论，Twitter/Weibo，tags云 或者自己能想到的差不多都能定制，就看网上能不能找到现成的或者自己能不能写了。
为了使界面简洁点，就只添加了一个分类列表

- 新建分类列表 source/_inclide/custom/asides/category_list.html

```javascript
<section>
  <h1>Categories</h1>
  <ul id="categories">
    {% category_list %}
  </ul>
</section>
```

- 新建插件： plugins/category_list_tag.rb

<!-- more -->

{% gist 3485442 category_list_tag.rb %}

-  更改配置 _config.yml

```yaml
default_asides: [asides/category_list.html, asides/recent_posts.html]
```


同时把侧边栏默认显示在最下面，保留可以在侧面打开的功能，页面布局看起来就简单多了
