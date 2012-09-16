---
layout: post
title: "Mac设置goagent开机自启动"
date: 2012-08-25 12:19
comments: true
categories: [mac, shell]
---
每次开机都得手动开启goagent，首先打开iTerm再输入，虽然平时很多时间都在跟终端打交道，也定义了快捷键go简单输入，但还是觉得麻烦，占了一个标签页不说，还时不时掉线，这个就无法忍了
搜索了下自动开启的方法，都是一些 linux 下的方法，比如说这两篇关键字靠前的

<!--more-->

- [开机自动启动GoAgent](http://keating.iteye.com/blog/1463521)
- [ubuntu下goagent开机自启动](http://adelzhang.blogspot.com/2011/10/ubuntugoagent.html)

用linux的同学可以参考下，对症下药，应该问题不大，不过这些方法在Mac下用不了，启动文件存放位置方法和linux还是有所不同的，那么有没有在Mac下自启动的简单方法呢，答案当然是有的，过程也不复杂
用过 [Homebrew](http://mxcl.github.com/homebrew/) (Mac下的一个包管理器) 的同学都会觉得这个工具很方便，相当于linux上的apt-get和yum，方便大家下载安装各种需要很多依赖的东西，可能有时候依赖包还不是很完善，但是差不多已经取代Macports成为Mac上最好用的包管理器了，没有用的建议装一个(和port只能2选1)

如果你安装过mysql/mongo这类可以有开机启动项的软件，在安装完之后会有一大堆提示，比如这样

```c
To launch on startup:
* if this is your first install:
mkdir -p ~/Library/LaunchAgents
cp /usr/local/Cellar/mysql/5.5.25a/homebrew.mxcl.mysql.plist ~/Library/LaunchAgents/
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist

* if this is an upgrade and you already have the homebrew.mxcl.mysql.plist loaded:
launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
cp /usr/local/Cellar/mysql/5.5.25a/homebrew.mxcl.mysql.plist ~/Library/LaunchAgents/
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist

You may also need to edit the plist to use the correct "UserName".
```

这个就是提示你怎么设置开机启动，brew已经预先把启动文件都写好了，你只需要copy过去，设置启动就Ok了。~/Library/LaunchAgents 下存放用户登陆之后的启动服务，具体补课可以参考 [这篇](http://kenwublog.com/mac-os-launchd-tuning)
有人做了个dmg包和goagent源码配合使用，把那个app设置为开机启动也很方便，位于 `系统偏好设置 > 用户与群组 > 登陆项` 参考 [这](http://dharmasong.net/2011/11/449.html)，这个应该是不爱折腾的最简单的方法

下面开始小折腾的方法，也很简单

- 新建一个plist文件，比如说叫 com.google.goagent.plist，输入以下内容

{% gist 3461688 com.google.goagent.plist %}


代码很简单，书写要规范。设置RunAtLoad和KeepAlive为true，试用了一天感觉良好

- 接下来敲几个字就over了

``` c
	cp com.google.goagent.plist ~/Library/LaunchAgents
	launchctl load -w ~/Library/LaunchAgents/com.google.goagent.plist
```

这样下次开机的时候goagent就已经自动启动了，总结起来就是三步:
写一个启动文件 ------ copy到对应目录 ------ 设为启动

默认开启8087端口

```c
lsof -i:8087
Python xxxxxx TCP localhost:8087 (LISTEN)
```
用Chrome或者Firefox用proxy switch已经配置好的同学可以直接打开敏感词测试了

停止该服务的方式也很简单，把这个 plist 移除当前目录 或者

```
# -w 参数是完全删除的意思，否则在下次登陆的时候依然会启动服务
launchctl unload -w ~/Library/LaunchAgents/com.google.goagent.plist
```

执行这2个操作中的一个后服务将会立即停止，需要注意的是，不能通过 kill PID 的方式杀死该服务，kill -9 删除服务也会重启，只能通过这两种方式才能终止，如果想修改服务最好先 unload 服务，修改完之后再 load，直接修改可能会导致电脑重启


