---
layout: post
title: "免去sudo时输入密码"
date: 2012-08-20 14:16
comments: true
categories: [mac]
---
经常敲命令的同学肯定会遇到需要root权限的问题，为了安全考虑大部分情况下千万不要在root权限下操作，否则会遇到各种难缠的问题，sudo 就是为解决这个问题产生的。

###为什么不想敲密码
<!-- more -->
- 程序员往往都是很懒的，很多代码的产生就是为了方便懒惰的程序员
- 输入密码很麻烦，很费时。(好吧，其实也花不了多长时间。。)
- 能方便设置一些启动项，特别是一个项目启动项很多，为了懒就写一个启动脚本
- 自己的系统自己用，没必要像在服务器上对权限要求那么严格
- 当然也有忘记root密码的情况。。

### 怎么做
其实很简单，只需要小修改 /etc/sudoers 这个文件就可以了。
默认情况的权限是这样的

    root ALL=(ALL) ALL
    %admin ALL=(ALL) ALL

意思是root用户和admin群组的用户都能执行所有命令，当然也是需要输入password的
修改如下：

    # 修改amdin组都不用输入密码
    %admin ALL=(ALL) NOPASSWD: NOPASSWD ALL
    # 只是想让 marine 用户输入sudo不需要密码
    marine ALL=(ALL) NOPASSWD: ALL

当然可定制丰富的权限要求，不过那样还不如不设置，统一要求输入密码得了。

###一个好用的小技巧
有时候当我们输入了一大串命令后敲下去发现需要写sudo，这个时候不得不找回上个命令，然后回到命令开头，然后加上sudo空格，稍微有点麻烦，可以用以下2个小方法

    sudo !!   	# !! 表示上次执行的命令

或者用zsh的同学可以自定义个命令绑定快捷键，比如这样:

	sudo-command-line() {
		[[ -z $BUFFER ]] && zle up-history
		[[ $BUFFER != sudo\ * ]] && BUFFER="sudo $BUFFER"
       zle end-of-line
	}
	zle -N sudo-command-line
	bindkey "\e\e" sudo-command-line

定义快捷键: 连续按2个ESC自动在前面添加sudo，非常方便快捷

前段时间看了个介绍 [linux命令的网站](http://www.commandlinefu.com/commands/browse/sort-by-votes)，有很多实用方便的快捷键，大家有兴趣可以去淘几个
