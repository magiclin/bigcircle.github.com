---
layout: post
title: "编译Textmate"
date: 2012-08-28 23:43
comments: true
categories: [mac, tools]
---
前段时间 [Textmate](http://macromates.com/) 开源着实让大家大吃一斤，论坛上，Twitter，包括微博上也都是各种转发，谈论。

Textmate 无须多作介绍，知道的应该都知道。号称能够和 Emacs Vim 并称的神器，当然是在 Mac 上和 Ruby 界，很多编译库都是系统依赖的，作者可能也没想移植到 linux/windows 下，所以没造成大火的局面，不过美金也赚的差不多了

<!--more-->

我从做开发伊始就一直用的 Sublime Text 2，也是一个大量借鉴 Textmate 理念和功能的产品，易用性相当不错，也有很多现成的包可以用，跨平台，不爽的是快捷键不能跨，windows下快捷键得另改，好在现在已经不需要切到 windows 了，也就少了这个烦恼。上次在 v2ex 看到有人发编译后的app，感觉编译应该不是很难。以前一直想试用下，但是version 1下的汉字显示实在是不行，现在升级到版本2了貌似解决了这个问题，就尝试着编译个尝尝鲜，看和 ST2 哪个更好用

Github 上托管地址 [Here](https://github.com/textmate/textmate)

#####安装编译所需工具

```c
brew install ragel boost multimarkdown hg ninja
```

boost 源码放在 sourceforge 上，下载可能需要挂代理，而且速度也很慢
ninja 最近才新加到 brew 的 Formula 里面，如果安装提示没有的话需要 brew update 下

#####查看是否有 pgrep pkill

```
which pgrep pkill
# 如果没有的话需要安装 proctools
brew install proctools
```

#####需要clang 3.2 / 4.0 版本

一般安装了 xcode 都已经安装好了 clang，直接查看版本

```
clang -v

Apple clang version 4.0 (tags/Apple/clang-421.0.60) (based on LLVM 3.1svn)
Target: x86_64-apple-darwin12.1.0
Thread model: posix
```

```
# 没有的话安装 llvm 套件
brew install --HEAD llvm --with-clang
```

##### start compile

```
git clone https://github.com/textmate/textmate.git
cd textmate
git submodule update --init
./configure && ninja
```

作者用 ninja 取代了 make 进行编译和安装

编译完了图标居然变成了菊花。。以前的图标差不多成为了一个标志，实在是理解不能。。

生成的 app 文件位于

```
~/build/TextMate/Applications/TextMate
```

图标就是那朵粉嫩的菊花。。难看死了

最后贴上一张芙蓉照

![](http://m1.img.libdd.com/farm5/2012/0829/00/00A980D9C6A159F9467E71EFCE4485420E266C05049E_1190_714.PNG)



