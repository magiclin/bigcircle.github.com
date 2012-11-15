---
layout: post
title: "修改 GC 参数提高 Ruby 性能"
date: 2012-11-15 15:50
comments: true
categories: ruby
keywords: ruby gc
description: improve ruby gc performance
---
最近看到有人提到最新的 ruby-patch 对性能有个很大的优化，不知是真是假，参考了老外一篇对 rvm 升级的[介绍](http://astrails.com/blog/2012/11/13/rvm-install-patched-ruby-for-faster-rails-startup)，在本机上测试了下，感觉没什么大的变化，可能本机性能比生产环境要好很多，这点小的优化在单次测试中体现不出来，但在服务器上可能就有很大的性能提升。在此介绍下，希望能提供点帮助

- 更新 rvm 到最新版本获取最新的 patch

```ruby
rvm get head
rvm install 1.9.3 --patch falcon -n falcon
```
<!--more-->
如果直接给 1.9.3 打补丁可能会爆 `Patch 'falcon' not found` 的错误，这是因为 falcon 补丁还没有支持到最新的 ruby 版本，需要在本地找到支持的最新版本

```
➜ ls $rvm_path/patches/ruby/1.9.3/*/*falcon* | sort
/Users/dayuan/.rvm/patches/ruby/1.9.3/p0/falcon.patch
/Users/dayuan/.rvm/patches/ruby/1.9.3/p125/falcon.patch
/Users/dayuan/.rvm/patches/ruby/1.9.3/p194/falcon.diff
/Users/dayuan/.rvm/patches/ruby/1.9.3/p286/falcon.diff
```

可以看到最新的补丁支持到p286版本，刚好本地原装的就是这个版本

```
➜ rvm install ruby-1.9.3-p286 --patch falcon -n falcon
# 安装完之后会重新编译一个ruby版本，而不是把原有版本覆盖
➜ ls ~/.rvm/rubies
default ruby-1.8.7-p370  ruby-1.9.3-p194  ruby-1.9.3-p286-falcon
```

- 接着修改 GC 参数，添加到 .bashrc / .zshrc 中

```
export RUBY_HEAP_MIN_SLOTS=1000000
export RUBY_HEAP_FREE_MIN=500000
export RUBY_HEAP_SLOTS_INCREMENT=1000000
export RUBY_HEAP_SLOTS_GROWTH_FACTOR=1
export RUBY_GC_MALLOC_LIMIT=100000000
```

GC参数设置，具体解释看 [这篇](http://bbs.chinaunix.net/thread-3661069-1-1.html)，ruby 默认的GC参数都太小，上面的参数是 REE 官方建议的参数

```
{
  RUBY_HEAP_MIN_SLOTS             => '初始堆大小，默认10000，越大需要占用的内存越多'
  RUBY_HEAP_FREE_MIN              => 'GC后可用的heap slot的最小值，默认4096，如果太小，就会按照下面2个参数分配新栈'
  RUBY_HEAP_SLOTS_INCREMENT       => '当Ruby需要开辟一片新的堆栈所需的数，默认是10000'
  RUBY_HEAP_SLOTS_GROWTH_FACTOR   => '当ruby需要新的堆栈的时候， 此参数做为一个乘数被用来计算这片新的堆栈的大小'
  RUBY_GC_MALLOC_LIMIT            => '允许不触发GC而分配的C数据结构的最大值，默认8000000 byte，设置的太低就会触发垃圾回收'
}
```

看下 37single 和 twitter 的参数设置，差距上差不多

```
37signals
RUBY_HEAP_MIN_SLOTS=600000 
RUBY_GC_MALLOC_LIMIT=59000000 
RUBY_HEAP_FREE_MIN=100000 

twitter
RUBY_HEAP_MIN_SLOTS=500000 
RUBY_HEAP_SLOTS_INCREMENT=250000 
RUBY_HEAP_SLOTS_GROWTH_FACTOR=1 
RUBY_GC_MALLOC_LIMIT=50000000 
```

- 接下来测试下执行情况

```
rvm use default
time `which rails` runner "puts :OK"
=> `which rails` runner "puts :OK"  0.57s user 0.04s system 99% cpu 0.610 total

rvm use ruby-1.9.3-p286-falcon
time `which rails` runner "puts :OK"
`which rails` runner "puts :OK"  0.14s user 0.02s system 98% cpu 0.163 total
```

可以看到执行时间从 0.610s 到 0.163s 提升了300%。。。。虽然是毫秒级别的，但这个提示貌似有点凶猛，等什么时候修改到生产环境测试下是不是效率提示真的有这么大
