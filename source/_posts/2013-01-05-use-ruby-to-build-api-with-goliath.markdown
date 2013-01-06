---
layout: post
title: "用 ruby 构建 api 之 goliath"
date: 2013-01-05 18:00
comments: true
categories: ruby
keywords: ruby api goliath grape
description: 
---
上期说到在 rails 中用 grape 方便构建 api，如果是日常简单运用还可以，但是一般需要api的地方都不是请求量很低的运用，或多或少都对并发量有所要求，用 rails 这种重型框架做可能就有点大材小用，而且并不是很符合高并发要求，很多慢查询会导致请求阻塞，有时候甚至丢失请求，这个就不 happy 了，也是不能容忍的，与其费时间优化代码，不如从框架层面找个更合适的选择
<!--more-->
有次在 beijing ruby 聚会时汉东哥分享了下他们用 goliath 做高并发 api 的经验，刚好最近也准备把 api 作为单独的服务从项目中独立出来，就用 goliath 尝试了下，效果很不错，简单介绍下 goliath

`goliath is an open source version of the non-blocking (asynchronous) ruby lightweight web server framework, powered by an eventMachine reactor`

异步非阻塞，基于 eventmachine，轻量级，易读(no collbacks)，易维护，听上去都是优点，let's start

上期已经把 grape 独立成了一个模块，为了不对现有 api 做太多更改，决定把 grape 整体迁移过来，目录结构如下

```ruby
├── Capfile
├── Gemfile
├── Rakefile
├── app
│   ├── api        # 以前 api 中的文件
│   ├── lib        # 其他接口的文件
│   └── models     # model
├── config
│   ├── application.rb  # 数据库连接
│   ├── config.yml      # 各种config都放在这个文件，所以没有单独一个 database.yml
│   ├── deploy.rb
│   └── listen.god      # god 配置文件
├── log
├── server.rb      # goliath 启动文件
└── tmp
```

由于还要用到 activerecord，所以就把原有项目中的 models 全部复制过来了，可以把里面无用代码全部删掉，因为所有的查询方法都在 api中。简单配置下 application.rb 和 server.rb，注意 config.yml 中 mysql adapter 最好用 em_mysql2

```
require 'em-synchrony/activerecord'
require 'yaml'
require 'mysql2'

env = ENV['DATABASE_URL'] ? 'production' : 'development'
DB_CONFIG = YAML.load_file(File.dirname(__FILE__) + '/config.yml')
mysql_config = DB_CONFIG['mysql'][env]
ActiveRecord::Base.establish_connection(mysql_config)
```

```
require "rubygems"
require "bundler/setup"
require 'goliath'
require 'grape'
require File.dirname(__FILE__) + "/app/api/api"
require File.dirname(__FILE__) + '/config/application'

# 初始化时加载路径 app/api app/lib app/models
%w[api models lib].each do |folder|
   $LOAD_PATH.unshift(File.expand_path(File.dirname(__FILE__) + "/app/#{folder}"))
end

# 自定义Log的输出样式
Goliath::Request.log_block = proc do |env, response, elapsed_time|
  method = env[Goliath::Request::REQUEST_METHOD]
  path = env[Goliath::Request::REQUEST_URI]

  env[Goliath::Request::RACK_LOGGER].info("#{response.status} #{method} #{path} in #{'%.2f' % elapsed_time} ms")
end

class ProjectApi < Goliath::API
  def response(env)
    ::Project::Api.call(env)
  end
end
```

注意在 api 中调用 models的时候需要 require model，所以需要在 api.rb 中require一下，这样在执行 Model 时才能顺利找到，如果报找不到文件的错误，需要把所有的 require 改成绝对路径，像下面一样

```
Dir[File.dirname(__FILE__) + '/../models/*.rb'].each {|file| require file}
```

如果不需要 goliath 提供的 middleware 的话简单配置下就可以运行了，虽然也是 rack-base，但是不能用基于 rack 规范的 web server 启动，按照官方说明是必须位于 Goliath::API 类中且不能自定义包含 run 的方法，那就只能单独开启一个进程了

```
# production 环境开始log 定向到 log/production.log
ruby server.rb -sv -e production -l log/production.log
```

配置下 god，开4个进程

```
# god 配置文件 监测 goliath 程序是否运行正常
# start => god -c config/listen.god -D

API_ENV = ENV['GOLIATH_ENV'] || 'production'
API_ROOT = "/path/to/goliath-api"
God.pid_file_directory = "#{API_ROOT}/config"
PROCESS_NUM = 4

(1..PROCESS_NUM).each do |port|
  God.watch do |w|

    w.dir = "#{API_ROOT}"
    w.log = "#{API_ROOT}/log/#{API_ENV}.log"
    port += 8000

    w.name = "api-#{port}"
    w.interval = 30.seconds # default
    w.start = "cd #{API_ROOT} && ruby server.rb -sv -e production -l log/production.log \
               -p #{port} -P #{API_ROOT}/tmp/pids/api.#{port}.pid -d"
    w.stop = "kill -QUIT `cat #{API_ROOT}/tmp/pids/api.#{port}.pid"
    w.restart = "#{w.stop} && #{w.start}"

    w.start_grace = 10.seconds
    w.restart_grace = 10.seconds     # 重启缓冲时间
    w.pid_file = "#{API_ROOT}/tmp/pids/api.#{port}.pid"

    w.behavior(:clean_pid_file)

    w.start_if do |start|
      start.condition(:process_running) do |c|
        c.interval = 5.seconds
        c.running = false
      end
    end

    w.restart_if do |restart|
      restart.condition(:memory_usage) do |c|
        c.above = 150.megabytes
        c.times = [3, 5] # 3 out of 5 intervals
        c.notify = 'your_name'      # 报警邮件发送对象
      end

      restart.condition(:cpu_usage) do |c|
        c.above = 50.percent
        c.times = 5
        c.notify = 'your_name'
      end
    end

    w.lifecycle do |on|
      on.condition(:flapping) do |c|
        c.to_state = [:start, :restart]
        c.times = 5
        c.within = 5.minute
        c.transition = :unmonitored
        c.retry_in = 10.minutes
        c.retry_times = 5
        c.retry_within = 2.hours
        c.notify = 'your_name'
      end
    end
  end
end
```

由于测试服务器上只是i5双核，nginx 开4个 worker_process 就可以了。派发本地 8001~8005端口，旧有的 api 就用 passenger 启动省事。注意要修改 hosts 文件，如果是本地测试就改成 `127.0.0.1 api_new.dev api_old.dev`，测试跑在测试服务器上也是修改本地 hosts，只是需要把 127.0.0.1 改成测试服务器的ip，这样 dns 才能正确解析，同时也能共用 80 端口

```
worker_processes 4

upstream api_cluster {
  server 127.0.0.1:8001;
  server 127.0.0.1:8002;
  server 127.0.0.1:8003;
  server 127.0.0.1:8004;
}

server {
  listen       80;
  server_name  api_new.dev;
  location / {
    proxy_pass http://api_cluster;
  }
}

server {
  listen       80;
  server_name  api_old.dev;
  location / {
    passenger_enabled on;
    root /path/to/rails-api/public;
  }
}
```

接着跑下 ab 测试，数据量以我们流量平均值3k为大小，并发2000，测试时间1分钟。如果想测试次数，需要先 -t 设置时间之后再 -n，这样可以设置一个超过 ab 最大测试数 50000 的值

```
# api_old
apr_socket_recv: Connection reset by peer (54)

# api_new
Requests per second:    6.44 [#/sec] (mean)
Time per request:       310788.789 [ms] (mean)
Time per request:       155.394 [ms] (mean, across all concurrent requests)
Transfer rate:          18.02 [Kbytes/sec] received

```

passenger 直接就扛不住了，goliath 虽然慢点，但也还是扛住了。最后测试了几次，最大并发在 2500 左右，这也跟测试环境机器性能，家里的渣路由有关。以正式环境32核的cpu及良好的带宽，应该能在这个基础上至少翻一倍。这个测试结果按照我们目前的流量还是比较满意的

最后介绍下god监控：如果需要 god 发送监控邮件，可以在 listen.god 中配置下smtp服务器和发送信息

```
# 以下用于发送监控邮件(进程未启动/内存超限/CPU超限)
God::Contacts::Email.defaults do |d|
  d.from_email = 'from_email'
  d.from_name = 'your_name'
  d.delivery_method = :smtp
  d.server_host = 'email_host'
  d.server_port = 25
  d.server_auth = :login
  d.server_domain = 'email_domail'
  d.server_user = 'email_address'
  d.server_password = 'your_email'
end

God.contact(:email) do |c|
 c.name = 'your_name'
 c.to_email = 'your_email'
end
```

参考

1. [0-60-with-goliath-building-high-performance-ruby-web-services](http://confreaks.com/videos/653-gogaruco2011-0-60-with-goliath-building-high-performance-ruby-web-services)
2. [grape-goliath-example](https://github.com/djones/grape-goliath-example)
3. [goliath wiki](https://github.com/postrank-labs/goliath/wiki) & [examples](https://github.com/postrank-labs/goliath/tree/master/examples)
4. [google mail group](https://groups.google.com/forum/?fromgroups#!forum/goliath-io)
