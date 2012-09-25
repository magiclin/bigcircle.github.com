---
layout: post
title: "一个好玩的东东--终端输出cow"
date: 2012-09-25 16:15
comments: true
categories: [mac, shell]
keywords: mac linux ruby shell
description: a funny command line tool
---
昨天看到 [lolcat](https://github.com/busyloop/lolcat)，本以为是 cat 的一个增强版，美化下输出什么的，搜了下才发现居然是用 ruby 写的，看了下源码，原来实现也很简单。代码虽少，版本号可不能少，已经迭代到42了，看来作者更新很勤快啊，还有一说是 42是宇宙的答案，或许作者为了获取最终的答案。先看下效果图

{% img /images/2012-09-25/1.png %}

安装方法也很简单

```c
brew install fortune
gem install cowsay lolcat
```
<!--more-->
其中 fotrune 负责随机生成一句话，cowsay 随机生成一个图像，lolcat 彩色打印
多试了几次发现生成的图像还挺多，于是好奇是怎么生成这些图像的，去 gem 里面看了下原来是预先定义了十九个图像，还以为真是随机生成的。。上面这个小恶魔的模板是

```ruby
module Cowsay
  module Character

    class Daemon < Base
      def template
        <<-TEMPLATE
   #{@thoughts}         ,        ,
    #{@thoughts}       /(        )`
     #{@thoughts}      \\ \\___   / |
            /- _  `-/  '
           (/\\/ \\ \\   /\\
           / /   | `    \\
           O O   ) /    |
           `-^--'`<     '
          (_.)  _  )   /
           `.___/`    /
             `-----' /
<----.     __ / __   \\
<----|====O)))==) \\) /====
<----'    `--' `.__,' \\
             |        |
              \\       /
        ______( (_  / \\______
      ,'  ,-----'   |        \\
      `--{__________)        \\/
        TEMPLATE
      end
    end

  end
end
```

这样只需要随机调用这些文件就可以生成样式了。再次启用谷歌娘查了下，原来这个灵感来自于 Linux 下的一个同名的东西叫做 cowsay / cowthink，下下来看下这个是怎么实现的

```
brew install cowthink
```

cowthink 其实就是在 cowsay 基础上封装了下，所以看 cowsay 就可以了

```
cd /usr/local/Cellar/cowsay/3.03/
find share/cows -name '*.cow' | wc -l
```

这个实现方式也是一样，先预设一些 cow，再随机输出，只不过这里面的 cow 多点，有46个，感兴趣的可以看下这里面还有哪些 cow

```
/usr/local/bin/cowsay -h
Usage: cowsay [-bdgpstwy] [-h] [-e eyes] [-f cowfile] 
          [-l] [-n] [-T tongue] [-W wrapcolumn] [message]

/usr/local/bin/cowsay -by -f daemon 'hello world' | lolcat
```
