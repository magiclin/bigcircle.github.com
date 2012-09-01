---
layout: post
title: "Sublime Text 2 使用 RVM "
date: 2012-09-01 13:43
comments: true
categories: [mac, ruby]
keywords: mac, sublime text 2, ruby
description: use rvm ruby in sublime text 2
---
用 Sublime Text 2 的同学使用 rvm 时会发现内置的可执行 ruby build 不可用了或者还是默认执行系统自带的 ruby-1.8.7 版本，当执行的脚本中用到 1.9.2 新语法的时候可能会被错，这个时候就需要切换到 rvm 版本控制的 ruby
<!--more-->
默认自带的 build 目录位于

```c
~/Library/Application\ Support/Sublime\ Text\ 2/Packages/Ruby/Ruby.sublime-build
```

默认的配置是这样的

```
"cmd": ["ruby", "$file"],
"file_regex": "^(...*?):([0-9]*):?([0-9]*)",
"selector": "source.ruby"
```

切换到 rvm 只需要改 cmd - ruby 指向路径， -KU 增加对中文输出支持，当然文件类型也要求是 utf-8

```
"cmd": ["/Users/yourname/.rvm/bin/rvm-auto-ruby", "-KU", "$file"]
```

你也可以自己改成 rvm 默认的 default ruby

```c
which ruby

=> /Users/yourname/.rvm/rubies/ruby-1.9.3-p194/bin/ruby
```

默认执行快捷键是 `⌘ + b` / `F7`，可以自己定义成喜欢的按键

```
{ "keys": ["super+b"], "command": "build" },
```

如何构建不用语言的 build，参考 [这篇](http://addyosmani.com/blog/custom-sublime-text-build-systems-for-popular-tools-and-languages/)，差不多列举全了，构造方式也很简单

------

另外推荐个我最常用的快捷键

- 在打开的几个标签之间前后切换。当打开很多标签时，又不喜欢用鼠标点来点去的可以试下这个

```
{ "keys": ["super+1"], "command": "prev_view" },
{ "keys": ["super+2"], "command": "next_view" },
```

这个快捷键同样适用于设置 Chrome/TotalFinder/iTerm 等能打开多标签的app，设置方式如下

![](http://m2.img.libdd.com/farm4/2012/0901/15/57756B60A70B4B4D6BEE07D177C25C55C856F5189977_500_448.jpg)

其他我定义的快捷键都在 [这](https://github.com/Bigcircle/config/blob/master/sublime/User/Default%20(OSX).sublime-keymap)，有需要的可以去淘几个

工欲善其事，必先利其器，利用好工具可以给开发带来很大的便利，节省很多时间