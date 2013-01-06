---
layout: post
title: "用 ruby 构建 api 之 grape"
date: 2013-01-03 17:08
comments: true
categories: ruby
keywords: ruby api grape
description: use ruby to create api
---
这几天看几篇文章介绍api的好处，刚好api也是我们业务中一块很重要的部分，最近也它作为一个单独的模块从主业务中独立出来，试用了下几个框架及不同的部署方案，虽然方法稍有差异，但是都很方便，只能说 ruby 在处理抽象业务逻辑上太方便了。首先介绍下 grape

<!--more-->

[grape](https://github.com/intridea/grape) is a rest-like api micro-framework for ruby. 最开始也是从 ruby-china 的源码中才知道它，看了下实现的确简单和方便，于是就开始把项目中的 api 全部迁移到了 grape 中。根据自己的业务逻辑相应的切分成了以下结构

```c
├── api.rb               # 主文件
├── entities.rb          # 一些简单查询调用
├── helpers.rb           # 一些公用方法
├── patch.rb             # 给 grape 打补丁(修改源码自用)
├── helpers              # 各个 api 文件复杂查询，相当于 controller 
│   ├── aa_helper.rb		
│   ├── bb_helper.rb
│   ├── cc_helper.rb
|   ├── ...
├── restfuls             # 路由存放文件，按照前缀放于不同文件，便于查找
│   ├── aa.rb
│   ├── bb.rb
|   ├── ...
|   ├── ...
└── validations          # 一些验证方法/信息
    ├── base.rb
    ├── depreated.rb     # 过期方法，为了兼容以前接口
    ├── error.rb         # 所有错误类型归类(因为自定义很多跟业务相关的错误)
    └── validate.rb
```

这样划分的目的在于使业务逻辑更清晰了，也便于维护，需要修改或添加什么 api 直接去 restfuls 中的对应文件修改就可以了。同时把以前在 model 中的代码都集中重到 helpers 文件中集中管理，修改也需要去 helpers 下对应的文件。这样差不多已经成了一个独立的模块，想拆分出去就很方便了

在 rails 中使用 grape 很方便，在 Gemfile 中添加 `gem grape` 后 `bundle install`，如果想放在 lib 下就直接在 lib 下新建文件夹把这些文件放进去，如果想放在 app 下就在 app 下新建文件夹放入这些文件，添加自动加载

```
(in config/application.rb)

class Application < Rails::Application
  ...
  config.autoload_paths += %W(#{config.root}/lib)
  config.autoload_paths += %W(#{config.root}/app/api)
end
```

接着在 route.rb 中挂载 api

```ruby
require 'api/api'
mount Project::Api => '/'
```

这样差不多所有的准备工作都完成了，只需要把所需要的接口单独写入对应的文件就可以了。写个测试 api

```ruby
Dir["#{Rails.root}/lib/api/restfuls/*.rb"].each {|file| require file}
Dir["#{Rails.root}/lib/api/*.rb"].each {|file| require file}

module Project
  class Api < Grape::API
    format :json                    
    helpers ApiHelpers              

    rescue_from :all do |e|
      Rack::Response.new([ e.message ], 500, { "Content-type" => "application/json" }).finish
    end

    mount Restfuls::Aa
    mount Restfuls::Bb

    params {requires :a, :type => String}
    get '/test(.:format)' do
      data = {:a => params[:a]}
      {:code => 200}.merge!(data).to_json
    end
  end
end
```

```ruby
curl http://localhost:3000/test.json?a=1
```

可能有人会有疑问，为什么不都写到 entities.rb 中而要分发到 helpers 文件夹中呢？取名 helpers 可能造成误解，其实它们充当的是 controller 的作用，里面的方法都是跟业务相关的方法。还有 entities 本身提供的功能太简单了，如果设计到复杂的 sql 语句或者需要根据需要定制输出信息，就会变得很麻烦，很多时候需要去 model 里面扩写 as_json 这个方法，还是用原生的写法比较舒服和便于理解修改。调用这些 helper 方法也很简单，和在 Rails 中一样，直接在对应的 restful/aa.rb 文件中 添加 `helpers Helpers::AaHelper` 就可以了，命名没有什么强制要求。

真正的 helper 方法都位于 helpers.rb 和 validations 中，可以自定义一些通用的方法和验证信息，如果需要引入外来接口也可以直接放在 helpers.rb 中，像我们会有个发短信的模块需要和 api 对接，可以将封装之后的接口直接拿过来生成一个调用方法，可能也就需要多添加几行代码，比如这样

```
(in helpers.rb)
require 'sms'    # lib/sms.rb 短信封装的thrift实现ruby client

module Project
  module ApiHelpers
  
    ...
    def send_message(t, c)
      Sms.new.start_send(t, c)
    end
  end
end
```

好像差不多就这么点东西，因为 grape 本身就是一个很简洁的 framework，grape wiki 内容也很齐全，基本上遇到的问题都能找到答案，如果不用重写代码的话，半天就能把原有代码全部牵过去，没有用过的同学可以试下。

##### Something

1. [为什么所有企业都需要API](http://www.ctocio.com/ccnews/3512.html)
2. [API利无穷，公司都该提供](http://tech2ipo.com/56814)
3. [grape wifi](https://github.com/intridea/grape/wiki)
4. [google group](https://groups.google.com/forum/?fromgroups#!forum/ruby-grape)