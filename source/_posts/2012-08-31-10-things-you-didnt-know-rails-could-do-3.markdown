---
layout: post
title: "你可能错过的 Rails 技巧(三)"
date: 2012-08-31 09:55
comments: true
categories: [rails]
keywords: rails, tricks
description: 10 things you didn't rails could do
---
中场休息时，去找了下 RailsConf 2012 的视频看了看，刚好看到关于 [这篇](http://confreaks.com/videos/889-railsconf2012-ten-things-you-didn-t-know-rails-could-do) 的介绍，片子还挺长，41分钟，演讲者长相和声音都很不符合大家对 Rails 的认知，大家有兴趣的可以去听听

###### 21 - 自动写文件

```ruby
class Comment < ActiveRecord::Base
  # ...
  Q_DIR = (Rails.root + "comment_queue").tap(&:mkpath)
  after_save :queue_comment
  def queue_comment
    File.atomic_write(Q_DIR + "#{id}.txt") do |f|
      f.puts "Article: #{article.subject}"
      f.puts "User:    #{user.name}"
      f.puts body
    end
  end
end
```

找了下 [Api](http://api.rubyonrails.org/) 是 Rails 对 Ruby 基础类的扩展

###### 22 - 合并嵌套 Hash

```
$ rails c
Loading development environment (Rails 3.2.3)
>> {nested: {one: 1}}.merge(nested: {two: 2})
=> {:nested=>{:two=>2}}
>> {nested: {one: 1}}.deep_merge(nested: {two: 2})
=> {:nested=>{:one=>1, :two=>2}}
```

主要是用到了 deep_merge 合并相同的 key
<!--more-->

###### 23 - Hash except

```
$ rails c
Loading development environment (Rails 3.2.3)
>> params = {controller: "home", action: "index", from: "Google"}
=> {:controller=>"home", :action=>"index", :from=>"Google"}
>> params.except(:controller, :action)
=> {:from=>"Google"}
```

这个方法经常会用到，可能用的人也很多

###### 24 - add defaultss to Hash

```
$ rails c
Loading development environment (Rails 3.2.3)
>> {required: true}.merge(optional: true)
=> {:required=>true, :optional=>true}
>> {required: true}.reverse_merge(optional: true)
=> {:optional=>true, :required=>true}
>> {required: true, optional: false}.merge(optional: true)
=> {:required=>true, :optional=>true}
>> {required: true, optional: false}.reverse_merge(optional: true)
=> {:optional=>false, :required=>true}
```

这几个都是对 Hash 类的增强，merge 会替换原有相同 key 的值，但 reverse_merge 不会

从源码就可以看出，会事先 copy 一份 default hash

```
def reverse_merge(other_hash)
  super
  self.class.new_from_hash_copying_default(other_hash)
end
```

###### 25 - String.value? 方法

看下面的几个例子

```
$ rails c
Loading development environment (Rails 3.2.3)
>> env = Rails.env
=> "development"
>> env.development?
=> true
>> env.test?
=> false
>> "magic".inquiry.magic?
=> true
>> article = Article.first
=> #<Article id: 1, ..., status: "Draft">
>> article.draft?
=> true
>> article.published?
=> false
```

env, "magic" 可以直接使用 value? 的方法，这个扩展是 String#inquiry 方法

```
def inquiry
  ActiveSupport::StringInquirer.new(self)
end

# 用method_missing 实现
def method_missing(method_name, *arguments)
  if method_name.to_s[-1,1] == "?"
    self == method_name.to_s[0..-2]
  else
    super
  end
end
```

类型的一个例子，同样用到了 inquiry 方法

```
class Article < ActiveRecord::Base
  # ...
  STATUSES = %w[Draft Published]
  validates_inclusion_of :status, in: STATUSES
  def method_missing(method, *args, &block)
    if method =~ /\A#{STATUSES.map(&:downcase).join("|")}\?\z/
      status.downcase.inquiry.send(method)
    else
      super
    end
  end
end
```

###### 26 - 让你成为杂志的封面 （暖场之用）

搞笑哥拿出了 DHH 当选 Linux journal 杂志封面的图片，会场也是哄堂大笑 ^.^

![](http://www.rubyinside.com/wp-content/uploads/2006/05/dhh.png)

###### 27 - 隐藏用户评论

```
<!-- HTML comments stay in the rendered content -->
<%# ERb comments do not %>
<h1>Home Page</h1>
# 生成的 html
<body>
<!-- HTML comments stay in the rendered content -->
<h1>Home Page</h1>
</body>
```

这个一下没看懂。。试了下项目里面的代码，原来是隐藏的意思。。

###### 28 - 理解更短的 erb 语法

```
# ...
module Blog
  class Application < Rails::Application

    # Broken:  config.action_view.erb_trim_mode = '%'
    ActionView::Template::Handlers::ERB.erb_implementation =
      Class.new(ActionView::Template::Handlers::Erubis) do
        include ::Erubis::PercentLineEnhancer
      end
    ￼￼￼￼end
  end
end
```

接着在 view 页面替换用 % 表示原来 <% if %> <% end %>，有点像 slim

```
% if current_user.try(:admin?)
  <%= render "edit_links" %>
% end
```

###### 29 - 用 block 避免视图层赋值

```
<table>
  <% @cart.products.each do |product| %>
    <tr>
      <td><%= product.name %></td>
      <td><%= number_to_currency product.price %></td>
    </tr>
  <% end %>
  <tr>
    <td>Subtotal</td>
    <td><%= number_to_currency @cart.total %></td>
  </tr>
  <tr>
    <td>Tax</td>
    <td><%= number_to_currency(tax = calculate_tax(@cart.total)) %></td>
  </tr>
  <tr>
    <td>Total</td>
    <td><%= number_to_currency(@cart.total + tax) %></td>
  </tr>
</table>
```

`tax = calculate_tax(@cart.total)` 会先被赋值再被下面引用

用 block 重构下，把逻辑代码放到 helper 里面

```
module CartHelper
  def calculate_tax(total, user = current_user)
    tax = TaxTable.for(user).calculate(total)
    if block_given?
      yield tax
    else
      tax
    end
  end
end
```

```
<table>
  <% @cart.products.each do |product| %>
    <tr>
      <td><%= product.name %></td>
      <td><%= number_to_currency product.price %></td>
    </tr>
  <% end %>
  <tr>
    <td>Subtotal</td>
    <td><%= number_to_currency @cart.total %></td>
  </tr>
  <% calculate_tax @cart.total do |tax| %>
    <tr>
      <td>Tax</td>
      <td><%= number_to_currency tax %></td>
    </tr>
    <tr>
      <td>Total</td>
      <td><%= number_to_currency(@cart.total + tax) %></td>
    </tr>
  <% end %>
</table>
```

###### 30 - 同时生成多个标签

```
<h1>Articles</h1>
  <% @articles.each do |article| %>
    <%= content_tag_for(:div, article) do %>
    <h2><%= article.subject %></h2>
  <% end %>
<% end %>
```

```
<%= content_tag_for(:div, @articles) do |article| %>
  <h2><%= article.subject %></h2>
<% end %>
```

content_tag_for 具体用法可以参考 [Api](http://api.rubyonrails.org/classes/ActionView/Helpers/RecordTagHelper.html#method-i-content_tag_for)，意思比较明白
