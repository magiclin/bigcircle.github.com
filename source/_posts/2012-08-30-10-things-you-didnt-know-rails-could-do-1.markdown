---
layout: post
title: "你可能错过的 Rails 技巧(一)"
date: 2012-08-30 11:22
comments: true
categories: [rails]
keywords: rails tips
description: introduce 10 things you didn't know rails could do
---
记得前段时间 RailsConf2012 之后看过一个不错的pdf，[10 things you didn't know rails could do](https://speakerdeck.com/u/jeg2/p/10-things-you-didnt-know-rails-could-do)

说是10个，但是给出了42个实例，这几天抽空又回味了下，料很多，写的很好，顺便总结学习下

Pass 掉第一个 [fridayhug](http://fridayhug.com)，我们是开心拥抱每一天
<!--more-->

###### 1 - 最小的rails app

```ruby
%w(action_controller/railtie coderay markaby).map &method(:require)

run TheSmallestRailsApp ||= Class.new(Rails::Application) {
  config.secret_token = routes.append {
    root to: proc {
      [200, {"Content-Type" => "text/html"}, [Markaby::Builder.new.html {
        title @title = "The Smallest Rails App"
        h3 "I am #@title!"
        p "Here is my source code:"
        div { CodeRay.scan_file(__FILE__).div(line_numbers: :table) }
        p { a "Make me smaller", href: "//goo.gl/YdRpy" }
      }]]
    }
  }.to_s
  initialize!
}
```

###### 2 - 提醒功能 TODO

```
class UsersController < ApplicationController
  # TODO:  Make it possible to create new users.
end

class User < ActiveRecord::Base
  # FIXME: Should token really  be accessible?
  attr_accessible :bil, :email, :name, :token
end

<%# OPTIMIZE: Paginate this listing. %>
<%= render Article.all %>
```

执行 `rake notes`

```
app/controllers/users_controller.rb:
  * [ 2] [TODO] Make it possible to create new users.

app/models/user.rb:
  * [ 2] [FIXME] Should token really be accessible?

app/views/articles/index.html.erb:
  * [ 1] [OPTIMIZE] Paginate this listing.
```

查看单独的 TODO / FIXME / OPTIMIZE 

```
rake notes:todo

app/controllers/users_controller.rb:
  * [ 2] Make it possible to create new users.
```

可以自定义提醒名称

```
class Article < ActiveRecord::Base
  belongs_to :user
  attr_accessible :body, :subject
  # JEG2: Add that code from your blog here.
end
```
不过需要敲一长串，可以alias个快捷键

```
rake notes:custom ANNOTATION=JEG2

app/models/article.rb:
  * [ 4]Add that code from your blog here.
```

###### 3 - 沙箱模式执行 rails c

```
rails c --sandbox
```

沙箱模式会有回滚事务机制，对数据库的操作在退出之前都会自动回滚到之前未修改的数据

###### 4 - 在 rails c 控制台中使用 rails helper 方法

```
$ rails c
Loading development environment (Rails 3.2.3)
>> helper.number_to_currency(100)
=> "$100.00"
>> helper.time_ago_in_words(3.days.ago)
=> "3 days"
```

###### 5 - 开发模式用 thin 代替 webrick

```
group :development do
  gem 'thin'
end

rails s thin / thin start
```

###### 6 - 允许自定义配置

```
 - lib/custom/railtie.rb
 
 module Custom
   class Railtie < Rails::Railtie
     config.custom = ActiveSupport::OrderedOptions.new
   end
 end
 
 - config/application.rb
 
 require_relative "../lib/custom/railtie"

 module Blog
   class Application < Rails::Application
     # ...
     config.custom.setting = 42
   end
 end
```

###### 7 - keep funny

作者给出了个介绍 ruby 以及一些相关 blog的网站 [rubydramas](http://www.rubydramas.com)，搞笑的是这个网站右上角标明 

```
Powered by PHP
```

用 [isitrails.com](http://isitrails.com) 检查了下，果然不是用 rails 做的，这个应该是作者分享 ppt 过程中的一个小插曲吧


###### 8 -理解简写的迁移文件

```
rails g resources user name:string email:string token:string bio:text
```

字段会被默认为 string 属性，查看了下 [源码](https://github.com/rails/rails/blob/master/railties/lib/rails/generators/generated_attribute.rb#LC55)，果然有初始化定义

```
rails g resources user name email token:string{6} bio:text
```

会生成用样的 migration 文件

```
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
      t.string :email
      t.string :token, :limit => 6
      t.text :bio
      t.timestamps
    end
  end
end
```

###### 9 - 给 migration 添加索引

```
rails g resource user name:index email:uniq token:string{6} bio:text
```

会生成 name 和 email 的索引，同时限定 email 唯一

```
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
      t.string :email
      t.string :token, :limit => 6
      t.text :bio
      t.timestamps
    end
    
    add_index :users, :name
    add_index :users, :email, :unique => true
  end
end
```

###### 10 - 添加关联关系

```
rails g resource article user:references subject body:text
```

会自动关联生成对应的 belongs_to 和 外键，并添加索引

```
class CreateArticles < ActiveRecord::Migration
  def change
    create_table :articles do |t|
      t.references :user
      t.string :subject
      t.text :body
      t.timestamps
    end
    add_index :articles, :user_id
  end
end
```

```
class Article < ActiveRecord::Base
  belongs_to :user
  attr_accessible :body, :subject
end
```

未完待续。。。太长了，还是分期吧