---
layout: post
title: "rails view select 方法学习"
date: 2012-08-28 10:52
comments: true
categories: [rails]
---
开始几篇都是扯些闲蛋，从这篇开始多写点技术方面学习的东西，多谈点实际，少谈点主义，做只勤劳的小蜜蜂

rails的actionview提供了简单的select方法生产表单选择项，根据 [Api](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html) 指示，用法如下：

```ruby
select(object, method, choices, options = {}, html_options = {})
```
<!--more-->
- object是指select选项所修饰的目标对象，通常是一个Model对象
- method是目标对象的属性（方法）名
- choices可以是任何可枚举的对象，数组，Hash或者是包含了选择框的数据库查询结果
- options选项
- html_options是html相关选项

include_blank 会显示值为空的默认选项，prompt 会给个提示选择，比如提示 Select One.   
例如对于  @post.person_id => 2  

```
select("post", "person_id", Person.all.collect {|p| [ p.name, p.id ] }, {:include_blank => 'None'})

<select name="post[person_id]">
  <option value="">None</option>
  <option value="1">David</option>
  <option value="2" selected="selected">Sam</option>
  <option value="3">Tobias</option>
</select>
```

index => nil 不显示空选项或提示项，直接显示第一个值

```
select("album[]", "genre", %w[rap rock country], {}, { :index => nil })

<select name="album[][genre]" id="album__genre">
  <option value="rap">rap</option>
  <option value="rock">rock</option>
  <option value="country">country</option>
</select>
```

:disabled => value 设置一个单独的值或者Prco对象 html标签属性为disable

```
select("post", "category", Post::CATEGORIES, {:disabled => 'restricted'})

<select name="post[category]">
  <option></option>
  <option>joke</option>
  <option>poem</option>
  <option disabled="disabled">restricted</option>
</select>
```

当用到collection_select时，可以鉴定一个Proc对象是否disable

```
collection_select(:post, :category_id, Category.all, :id, :name, {:disabled => lambda{|category| category.archived? }})

<select name="post[category_id]">
  <option value="1" disabled="disabled">2008 stuff</option>
  <option value="2" disabled="disabled">Christmas</option>
  <option value="3">Jokes</option>
  <option value="4">Poems</option>
</select>
```

html_option 里面还可以写各种js事件，比如这样

```
select(:person, :id, Persion.all, {:onchange => 'doSomething()'})
```

##### [select_tag](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html#method-i-select_tag)

```
select_tag(name, option_tags = nil, options = {})
```

option => {:multiple, :disable, :include_blank, :prompt} 
后三个和select里面用法一样，mutiple 允许同时传递多个值，相当于一个多选框

option_tags 可以自己手写几个option标签，或者用现成的方法，其实就是option标签的helper方法，Api 中的几个 [例子](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-options_for_select)

```
options_for_select(container, selected = nil)

options_from_collection_for_select(collection, value_method, text_method, selected = nil)
```

##### [collection_select](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_select)

```
collection_select(object, method, collection, value_method, text_method, options = {}, html_options = {})
```

collect_select 比 select 多了2个选项，value_method 和 text_method 分别表示 collection 你想选择的对应字段，相当于 select + options_from_collection_for_select 的组合，大致功能其实和 select 差不多

一篇老外5年前写的介绍这几个 select 的 [文章](http://shiningthrough.co.uk/Select-helper-methods-in-Ruby-on-Rails)，rails 的中文文档还是太少了。

剩下几个出场率太低的helper

```
grouped_collection_select(object, method, collection, group_method, group_label_method, option_key_method, option_value_method, options = {}, html_options = {})

grouped_options_for_select(grouped_options, selected_key = nil, prompt = nil)

option_groups_from_collection_for_select(collection, group_method, group_label_method, option_key_method, option_value_method, selected_key = nil)
```

- grouped_collection_select用法和 collection_select 差不多，只是会生成 optgroup 标签来标识二级选项
- grouped_options_for_select 也是一样，比 options_for_select 多一层 optgroup 标签
- option_groups_from_collection_for_select 同options_from_collection_for_select，都只是在原有基础上修改了下

Rails 源码在此，有兴趣的可以拜读下 [Here](https://github.com/rails/rails/blob/27c8debdc6b242c845a279187205a2b057e18469/actionpack/lib/action_view/helpers/form_options_helper.rb#L156)

用 select 的好处就是书写简洁，可以配合js生产联动查询，比如说最常用的省市查询   
不过对于不熟悉语法的人可能读起来就不如直接 select > option 直接明了，需要花时间去明白什么意思，各有利弊吧