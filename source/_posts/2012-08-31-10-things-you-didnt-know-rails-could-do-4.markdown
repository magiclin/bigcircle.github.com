---
layout: post
title: "你可能错过的 Rails 技巧(四)"
date: 2012-08-31 14:47
comments: true
categories: [rails]
keywords: rails tricks
description: 10 things you didn't rails could do
---
这个系列完结篇，貌似内容挺多，但是实用的没几个

从阅读心里学来说，太长的篇幅容易让读者失去忍耐力，文章后面的内容就会草草略过，怎么感觉太长的系列写起来也让我失去忍耐力呢。。。继续忍耐，完结此篇

这篇 [blog](http://blog.plataformatec.com.br/2012/01/my-five-favorite-hidden-features-in-rails-3-2/) 介绍了4个最喜欢的 Rails3.2 隐藏特性，这4条都在这个系列中，作者可能也是从这学来的吧

###### 31 - Render Any Object

```ruby
class Event < ActiveRecord::Base
  # ...
  def to_partial_path
    "events/#{trigger}"  # events/edit or events/view
  end
end

<%= render partial: @events, as: :event %>
```

[to_partial_path](http://edgeapi.rubyonrails.org/classes/ActiveModel/Conversion.html#method-i-to_partial_path) 是 ActiveModel 內建的实例方法，返回一个和可识别关联对象路径的字符串，原文是这么说的，目前还没看明白这么用的目的在哪

```
Returns a string identifying the path associated with the object.
ActionPack uses this to find a suitable partial to represent the object.
```

###### 32 - 生成 group option

```
<%= select_tag( :grouped_menu, grouped_options_for_select(
  "Group A" => %w[One Two Three],
  "Group B" => %w[One Two Three]
) ) %>
```

这个其实就是用到了 grouped_options_for_select ，我在前面的 [博文](http://dayuan.im/blog/explain-select-on-rails-hepler.html/) 提到过这几个 select 的用法

<!--more-->

###### 33 -定制你自己喜欢的 form 表单

```
class LabeledFieldsWithErrors < ActionView::Helpers::FormBuilder
  def errors_for(attribute)
    if (errors = object.errors[attribute]).any?
      @template.content_tag(:span, errors.to_sentence, class: "error")
    end
  end
  def method_missing(method, *args, &block)
    if %r{ \A (?<labeled>labeled_)?
              (?<wrapped>\w+?)
              (?<with_errors>_with_errors)? \z }x =~ method and
       respond_to?(wrapped) and [labeled, with_errors].any?(&:present?)
      attribute, tags = args.first, [ ]
      tags           << label(attribute) if labeled.present?
      tags           << send(wrapped, *args, &block)
      tags           << errors_for(attribute) if with_errors.present?
      tags.join(" ").html_safe
    else
      super
    end
  end
end
```

定义了几个不想去看懂的 method_missing 方法。。
修改 application.rb，添加配置

```
class Application < Rails::Application
  # ...
  require "labeled_fields_with_errors"
  config.action_view.default_form_builder = LabeledFieldsWithErrors
  config.action_view.field_error_proc     = ->(field, _) { field }
end
```

创建 form 表单可以这样书写

```
<%= form_for @article do |f| %>
  <p><%= f.text_field
  <p><%= f.labeled_text_field
  <p><%= f.text_field_with_errors
  <p><%= f.labeled_text_field_with_errors :subject %></p>
  <%= f.submit %>
<% end %>
```

生成如下的 html 页面

```
<p><input id="article_subject" name="article[subject]" size="30" type="text" value="" /></p>
<p><label for="article_subject">Subject</label>
   <input id="article_subject" name="article[subject]" size="30" type="text" value="" /></p>
<p><input id="article_subject" name="article[subject]" size="30" type="text" value="" />
   <span class="error">can't be blank</span></p>
<p><label for="article_subject">Subject</label>
   <input id="article_subject" name="article[subject]" size="30" type="text" value="" />
   <span class="error">can't be blank</span></p>
<!-- ... -->
```

不是很喜欢这种方式，反而把简单的html搞复杂了，让后来维护的人增加额外的学习成本

###### 34 - Inspire theme songs about your work (再次暖场时刻)

2011年 Farmhouse Conf 上主持人 Ron Evans 专门用口琴演奏了为大神 Tenderlove 写的歌 - [Ruby Hero Tenderlove! ](http://www.confreaks.com/videos/529-farmhouseconf-ruby-hero-tenderlove)，听了半天不知道唱的啥。。

想找下有没有美女 Rubist, 看了下貌似没有，都是大妈，这位 [Meghann Millard](http://www.confreaks.com/videos/534-farmhouseconf-meghann-millard) 尚可远观，大姐装束妖娆，手握纸条，蚊蝇环绕，不时微笑，长的真有点像 gossip girl 里面的 Jenny Humphrey

###### 35 - 灵活的异常操作

修改 application.rb 定义

```
class Application < Rails::Application
# ...
  config.exceptions_app = routes
end
```
每次有异常时路由都会被调用，你可以用下面的方法简单 render 404 页面

```
match "/404", :to => "errors#not_found"
```

这个例子也在开头提到的那篇博文里面，感兴趣可以去自己研究下

###### 36 - 给 Sinatra 添加路由

```
- Gemfile

source 'https://rubygems.org'
# ...
gem "resque", require: "resque/server"



module AdminValidator

  def matches?(request)
    if (id = request.env["rack.session"]["user_id"])
      current_user = User.find_by_id(id)
      current_user.try(:admin?)
    else
      false
    end
  end
end
```

挂载 Resque::Server 至 /admin/resque

```
Blog::Application.routes.draw do
  # ...
  require "admin_validator"
  constraints AdminValidator do
    mount Resque::Server, at: "/admin/resque"
  end
end
```

这个也没有试验，不清楚具体用法，sinatra 平时也基本不用

######　37 - 导出CSV流

```
class ArticlesController < ApplicationController
  def index
    respond_to do |format|
      format.html do
        @articles = Article.all
      end
      format.csv do
        headers["Content-Disposition"] = %Q{attachment; filename="articles.csv"}
        self.response_body = Enumerator.new do |response|
          csv  = CSV.new(response, row_sep: "\n")
          csv << %w[Subject Created Status]
          Article.find_each do |article|
            csv << [ article.subject,
                     article.created_at.to_s(:long),
                     article.status ]
        ￼￼	end
        end
      end
    end
  end
# ...
end
```

导出 csv 是很常用的功能，很多时候报表都需要，这个还是比较实用的

###### 38 - do some work in the background

给 articles 添加文本类型 stats 字段

```
rails g migration add_stats_to_articles stats:text
```

添加一个计算 stats 方法 和 一个 after_create 方法，在创建一条记录后，会把 calculate_stats 添加到 Queue 队列，当队列中有任务时，后台创建一个线程执行该 job

```
class Article < ActiveRecord::Base
  # ...
  serialize :stats
  def calculate_stats
    words = Hash.new(0)
    body.to_s.scan(/\S+/) { |word| words[word] += 1 }
    sleep 10  # simulate a lot of work
    self.stats = {words: words}
  end

  require "thread"
  def self.queue; @queue ||= Queue.new end
  def self.thread
    @thread ||= Thread.new do
      while job = queue.pop
        job.call
      end
    end
  end
  thread  # start the Thread

  after_create :add_stats
  def add_stats
    self.class.queue << -> { calculate_stats; save }
  end
end
```

添加一条记录，10秒后会自动给该记录 stats 字段添加 words Hash

```
$ rails c
Loading development environment (Rails 3.2.3)
>> Article.create!(subject: "Stats", body: "Lorem ipsum...");
Time.now.strftime("%H:%M:%S")
=> "15:24:10"
>> [Article.last.stats, Time.now.strftime("%H:%M:%S")]
=> [nil, "15:24:13"]
>> [Article.last.stats, Time.now.strftime("%H:%M:%S")]
=>[{:words=>{"Lorem"=>1, "ipsum"=>1, ...}, "15:24:26"]
```

###### 39 - 用 Rails 生成静态站点

修改 config/environment/development.rb

```
Static::Application.configure do
  # ...
  # Show full error reports and disable caching
  config.consider_all_requests_local       = true
  config.action_controller.perform_caching = !!ENV["GENERATING_SITE"]
  # ...
  # Don't fallback to assets pipeline if a precompiled asset is missed
  config.assets.compile = !ENV["GENERATING_SITE"]
  # Generate digests for assets URLs
  config.assets.digest = !!ENV["GENERATING_SITE"]
  # ...
end


class ApplicationController < ActionController::Base
  protect_from_forgery
  if ENV["GENERATING_SITE"]
    after_filter do |c|
      c.cache_page(nil, nil, Zlib::BEST_COMPRESSION)
    end
  end
end
```

修改 rake static:generate 任务

```
require "open-uri"
namespace :static do
  desc "Generate a static copy of the site"
  task :generate => %w[environment assets:precompile] do
    site = ENV.fetch("RSYNC_SITE_TO") { fail "Must set RSYNC_SITE_TO" }
    server = spawn( {"GENERATING_SITE" => "true"},
                    "bundle exec rails s thin -p 3001" )
    sleep 10  # FIXME: start when the server is up

    # FIXME: improve the following super crude spider
    paths = %w[/]
    files = [ ]
    while path = paths.shift
      files << File.join("public", path.sub(%r{/\z}, "/index") + ".html")
      File.unlink(files.last) if File.exist? files.last
      files << files.last + ".gz"
      File.unlink(files.last) if File.exist? files.last
      page = open("http://localhost:3001#{path}") { |url| url.read }
      page.scan(/<a[^>]+href="([^"]+)"/) do |link|
        paths << link.first
      end
    end

    system("rsync -a public #{site}")

    Process.kill("INT", server)
    Process.wait(server)
    system("bundle exec rake assets:clean")
    files.each do |file|
      File.unlink(file)
    end
  end
end
```

生成到某个地方，去查看吧

```
rake static:generate RSYNC_SITE_TO=/Users/james/Desktop
```

后面几个都不感兴趣，没有测试，说好的42个，瞎扯了3个pass掉了，实在是吐血了

Over.
