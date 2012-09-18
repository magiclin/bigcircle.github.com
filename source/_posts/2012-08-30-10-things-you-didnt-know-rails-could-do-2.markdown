---
layout: post
title: "你可能错过的 Rails 技巧(二)"
date: 2012-08-30 21:39
comments: true
categories: [rails]
keywords: rails tips
description: 10 things you didn't know rails could do
---
接着上期说 [你可能错过的 Rails 技巧(一)](http://dayuan.im/blog/10-things-you-didnt-know-rails-could-do-1.html/)

###### 11 - 显示数据迁移记录

```ruby
rake db:migrate:status
```

会输出 migration 的状态，这在解决迁移版本冲突的时候很有用

```
database: db/development.sqlite3

 status   Migration ID    Migration Name
 ---------------------------------------
   up     20120414155612  Create users
   up     20120414160528  Create articles
  down    20120414161355  Create comments
```

###### 12 - 导入 csv 文件

csv 文件内容如下

```
Name,Email
James,james@example.com
Dana,dana@example.com
Summer,summer@example.com
```

创建 rake 任务导入 users 表

<!--more-->

```
require 'csv'
namespace :users do
  desc "Import users from a CSV file"
  task :import => :environment do
    path = ENV.fetch("CSV_FILE") {
      File.join(File.dirname(__FILE__), *%w[.. .. db data users.csv])
    }
    CSV.foreach(path, headers: true, header_converters: :symbol) do |row|
      User.create(row.to_hash)
    end
  end
end
```

###### 13 - 数据库中储存 csv

```
class Article <  ActiveRecord::Base
  require 'csv'
  module CSVSerializer
    module_function
    def load(field); field.to_s.parse_csv; end
    def dump(object); Array(object).to_csv; end
  end
  serialize :categories, CSVSerializer

  attr_accessible :body, :subject, :categories
end
```

serialize 用于在文本字段储存序列化的值，如序列，Hash，Array等，例如

```
user = User.create(:preferences => { "background" => "black", "display" => large })
User.find(user.id).preferences # => { "background" => "black", "display" => large }
```

这个例子中将 CSVSerializer to_csv序列化之后储存在 categories 这个文本类型字段中

###### 14 - 用 pluck 查询

```
$ rails c
loading development environment(Rails 3.2.3)

>> User.select(:email).map(&:email)
  User Load(0.1ms) SELECT email FROM "users"
=> ["james@example.com", "dana@example.com", "summer@example.com"]
>> User.pluck(:email)
   (0.2ms) SELECT email FROM "users"
=> ["james@example.com", "dana@example.com", "summer@example.com"]
>> User.uniq.pluck(:email)
   (0.2ms) SELECT DISTINCT email FROM "users"
=> ["james@example.com", "dana@example.com", "summer@example.com"]
```

pluck 的实现方式其实也是 select 之后再 map

```
def pluck(column_name)
  column_name = column_name.to_s
  klass.connection.select_all(select(column_name).arel).map! do |attributes|
    klass.type_cast_attribute(attributes.keys.first, klass.initialize_attributes(attributes))
  end
end
```

###### 15 - 使用 group count

创建 article 关联 model event

```
rails g resource event article:belongs_to triggle
```

创建3条 edit 记录和10条 view 记录。 Event.count 标明有13条记录，
group(:triggle).count 表示统计 triggle group 之后的数量，也可以用 count(:group => :trigger)


```
$ rails c

>> article = Article.last
=> #<Article id:1, ...>
>> {edit:3, view:10}.each do |trigger, count|
?>   count.times do
?>     Event.new(trigger: trigger).tap{ |e| e.article= article; e.save! }
?>   end
=> {:edit => 3, :view => 10}
>> Event.count
=> 13
>> Event.group(:trigger).count
=> {"edit" => 3, "view" => 10}
```


###### 16 - 覆盖关联关系

```
class Car < ActiveRecord::Base
  belongs_to :owner
  belongs_to :previous_owner, class_name: "Owner"

  def owner=(new_owner)
    self.previous_owner = owner
    super
  end
end
```

car更改 owner 时，如果有了 new_owner，就把原 owner 赋给 previous_owner，注意要加上 super

###### 17 - 构造示例数据

```
$ rails c
Loading development environment (Rails 3.2.3)
>> User.find(1)
=> #<User id: 1, name: "James", email: "
￼￼￼￼￼￼￼james@example.com", ...>
￼￼￼￼￼>> jeg2 = User.instantiate("id" => 1, "email" => "
￼￼￼￼=> #<User id: 1, email: "james@example.com">
>> jeg2.name = "James Edward Gray II"
￼￼￼￼=> "James Edward Gray II"
>> jeg2.save!
=> true
>> User.find(1)
￼￼￼￼￼￼james@example.com", ...>
```

伪造一条记录，并不是数据库中的真实数据，也不会把原有数据覆盖

###### 18 - PostgreSQL 中使用无限制的 string

去掉适配器中对 string 长度的限制，这个应该是 PostgreSQL 数据库的特性

```
module PsqlApp
  class Application < Rails::Application
    # Switch to limitless strings
    initializer "postgresql.no_default_string_limit" do
      ActiveSupport.on_load(:active_record) do
        adapter = ActiveRecord::ConnectionAdapters::PostgreSQLAdapter
        adapter::NATIVE_DATABASE_TYPES[:string].delete(:limit)
      end
    end
 end
end
```

创建 user 表，bio 字符串

```
rails g resource user bio
```

```
$ rails c
Loading development environment (Rails 3.2.3)
>> very_long_bio = "X" * 10_000; :set
=> :set
>> User.create!(bio: very_long_bio)
=> #<User id: 1, bio:
"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XX...", created_at: "2012-04-14 23:02:08",
updated_at: "2012-04-14 23:02:08">
>> User.last.bio.size
=> 10000
```

###### 19 - PostgreSQL 中使用全文搜索

```
rails g resource article subject body:text
```

更改迁移文件，添加索引和 articles_search_update 触发器

```
class CreateArticles < ActiveRecord::Migration
  def change
    create_table :articles do |t|
      t.string :subject
      t.text   :body
      t.column :search, "tsvector"
      t.timestamps
    end
    execute <<-END_SQL
      CREATE INDEX articles_search_index
      ON articles USING gin(search);
      CREATE TRIGGER articles_search_update
      BEFORE INSERT OR UPDATE ON articles
      FOR EACH ROW EXECUTE PROCEDURE
      tsvector_update_trigger( search, 'pg_catalog.english', subject, body);
    END_SQL
  end
end
```

Model 中添加 search 方法

```
class Article < ActiveRecord::Base
  attr_accessible :body, :subject
  def self.search(query)
    sql = sanitize_sql_array(["plainto_tsquery('english', ?)", query])
    where(
      "search @@ #{sql}"
    ).order(
      "ts_rank_cd(search, #{sql}) DESC"
    )
end end
```

PostgreSQL 数据库没用过，这段看例子吧

```
$ rails c
Loading development environment (Rails 3.2.3)
>> Article.create!(subject: "Full Text Search")
=> #<Article id: 1, ...>
>> Article.create!(body: "A stemmed search.")
=> #<Article id: 2, ...>
>> Article.create!(body: "You won't find me!")
=> #<Article id: 3, ...>
>> Article.search("search").map { |a| a.subject || a.body }
=> ["Full Text Search", "A stemmed search."]
>> Article.search("stemming").map { |a| a.subject || a.body }
=> ["A stemmed search."]
```

###### 21 - 每个用户使用不同的数据库

```
- user_database.rb

def connect_to_user_database(name)
  config = ActiveRecord::Base.configurations["development"].merge("database" => "db/#{name}.sqlite3")
  ActiveRecord::Base.establish_connection(config)
end
```

创建 rake 任务：新增用户数据库和创建

```
require "user_database"

namespace :db do
  desc "Add a new user database"
  task :add => %w[environment load_config] do
    name = ENV.fetch("DB_NAME") { fail "DB_NAME is required" }
    connect_to_user_database(name)
    ActiveRecord::Base.connection
  end

  namespace :migrate do
    desc "Migrate all user databases"
    task :all => %w[environment load_config] do
      ActiveRecord::Migration.verbose = ENV.fetch("VERBOSE", "true") == "true"
      Dir.glob("db/*.sqlite3") do |file|
        next if file == "db/test.sqlite3"
        connect_to_user_database(File.basename(file, ".sqlite3"))
        ActiveRecord::Migrator.migrate(
          ActiveRecord::Migrator.migrations_paths,
          ENV["VERSION"] && ENV["VERSION"].to_i
        ) do |migration|
          ENV["SCOPE"].blank? || (ENV["SCOPE"] == migration.scope)
        end
      end
    end
  end
end
```

执行几个rake 任务创建不同的数据库

```
$ rails g resource user name
$ rake db:add DB_NAME=ruby_rogues
$ rake db:add DB_NAME=grays
$ rake db:migrate:all
==  CreateUsers: migrating ==================================
-- create_table(:users)
   -> 0.0008s
==  CreateUsers: migrated (0.0008s) =========================
==  CreateUsers: migrating ==================================
-- create_table(:users)
   -> 0.0007s
==  CreateUsers: migrated (0.0008s) =========================
```

创建几条记录，为不同的数据库创建不同的数据

```
$ rails c
>> connect_to_user_database("ruby_rogues")
=> #<ActiveRecord::ConnectionAdapters::ConnectionPool...>
>> User.create!(name: "Chuck")
=> #<User id: 1, name: "Chuck", ...>
>> User.create!(name: "Josh")
=> #<User id: 2, name: "Josh", ...>
>> User.create!(name: "Avdi")
=> #<User id: 3, name: "Avdi", ...>
...
>> connect_to_user_database("grays")
=> #<ActiveRecord::ConnectionAdapters::ConnectionPool...>
>> User.create!(name: "James")
=> #<User id: 1, name: "James", ...>
>> User.create!(name: "Dana")
=> #<User id: 2, name: "Dana", ...>
>> User.create!(name: "Summer")
=> #<User id: 3, name: "Summer", ...>
```

ApplicationController 里面添加 before_filter 根据登陆的二级域名判断连接哪个数据库

```
class ApplicationController < ActionController::Base
  protect_from_forgery
  before_filter :connect_to_database
private
  def connect_to_database
    connect_to_user_database(request.subdomains.first)
  end
end
```

中场休息，未完待续。。。内容真心多啊，不过有用的不多
