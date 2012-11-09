---
layout: post
title: "使用 sublime 修改远程文件"
date: 2012-11-09 11:18
comments: true
categories: mac sublime
keywords: ruby sublime mac rsub
description: "use sublime to modify remote file"
---
遇到个特殊情况，服务器不能用 git 同步代码，如果遇到 bug 需要及时修复，可能一次更改不完善，需要改了再改。一般情况下我们都是在本地修改测试OK之后更新到服务器，不能在本地修改的情况下我们只能在服务器上直接修改了，但是对于一个不是很习惯用vim同时又很习惯sublime的猿来说就很淡腾了

如果能用sublime直接修改服务器上代码就好了，以前textmate就有过这样的先例，rmate就是创造出来做这个的，好在sublime很多时间都可以把textmate的东西直接拿过来用，rmate也不例外

- 装一个叫 rsub 的插件，直接在 install package里面搜就有
- 开启本地端口转发，rsub 默认的是 52698

```c
ssh -R 52698:localhost:52698 user@remote_domain
```
<!-- more -->
- 或者在 .ssh/config 中加入

```
host remote_domain
  RemoteForward 52698 127.0.0.1:52698
```

- 登陆服务器 ssh user@123.456.78.90，添加rmate脚本

```
curl https://raw.github.com/aurora/rmate/master/rmate > rmate
sudo mv rmate /usr/local/bin
sudo chmod +x /usr/local/bin/rmate

# 在 ~/.bash_profile里面定义快捷键
alias sl='sudo rmate'
```

我一般比较懒，会开启我使用的用户免sudo输入密码，自用服务器上能省则省。。。但是可能会有安全问题，使用要慎重。接着就可以在服务器上打开需要修改的文件

```
sl your_file.xx
# 如果遇到只读文件，可以添加 -f 参数修改
sl -f your_file.xx
```

但是这样只能在服务上打开一个文件到本地修改，修改完之后会自动同步到服务器上，和用 vim 修改的唯一区别就是可以使用本地的编辑器了。功能有限，不能提供目录树修改，只是为习惯编辑器的同学提供另一种选择而已。大家还是应该努力深入自觉勤奋地学习vim啊！！








