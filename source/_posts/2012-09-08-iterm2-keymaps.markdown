---
layout: post
title: "iterm2 快捷键介绍"
date: 2012-09-08 15:30
comments: true
categories: [mac]
keywords: mac iterm2 keymaps
description: iterm2 keymaps intro
---
Mac 原来自带的终端工具 Terminal 不好用是出了名的，虽然最近几个版本苹果稍微做了些优化，功能上，可用性方面增强不少，无奈有个更好用的 Iterm2 摆在那，基本上也就没有多少出场机会了

[Iterm2](http://www.iterm2.com/)，经常使用终端的同学肯定早就切换到这个东东上了，开源免费，和 zsh 搭配差不多已经取代 Terminal + bash 成了 Mac 上终端工具的标准配置。
<!--more-->
Iterm2 的优点：

- 兼容性好，远程服务器 vi 什么的低版本能很好兼容，Terminal 则会出问题
- 支持 xterm-256 色，方便在终端中配置 vim/emacs 代码配色
- 快捷键丰富，自带/自己定义都很方便
- 分屏简单方便，可以根据自己需要同时搭配上 tmux，大屏用起来爽到爆

[官方文档](http://www.iterm2.com/#/section/documentation) 有非常详细的介绍，先来看看自带有哪些很实用的功能/快捷键

1. ⌘ + 数字在各 tab 标签直接来回切换
2. 选择即复制 + 鼠标中键粘贴，这个很实用
3. ⌘ + f 所查找的内容会被自动复制
4. ⌘ + d 横着分屏 / ⌘ + shift + d 竖着分屏
5. ⌘ + r = clear，而且只是换到新一屏，不会想 clear 一样创建一个空屏
6. ctrl + u 清空当前行，无论光标在什么位置
7. 输入开头命令后 按 ⌘ + ; 会自动列出输入过的命令
8. ⌘ + shift + h 会列出剪切板历史
9. 可以在 `Preferences > keys` 设置全局快捷键调出 iterm，这个也可以用过 Alfred 实现 

我常用的一些快捷键

1. ⌘ + 1 / 2 左右 tab 之间来回切换，这个在 [前面](http://dayuan.im/blog/add-rvm-ruby-path-to-sublime-text-2.html/) 已经介绍过了 
2. ⌘← / ⌘→ 到一行命令最左边/最右边 ，这个功能同 C+a / C+e
3. ⌥← / ⌥→ 按单词前移/后移，相当与 C+f / C+b，其实这个功能在Iterm中已经预定义好了，⌥f / ⌥b，看个人习惯了
4. 好像就这几个。。囧

设置方法如下

![](http://m1.img.libdd.com/farm5/2012/0908/17/523CEF031371E3BC2A56926F002FA3E2E651F805049E_924_541.PNG)

当然除了这些可以自定义的也不能忘了 linux 下那些好用的组合

1. C+a / C+e 这个几乎在哪都可以使用
2. C+p / !! 上一条命令
3. C+k 从光标处删至命令行尾 (本来 C+u 是删至命令行首，但iterm中是删掉整行)
4. C+w A+d 从光标处删至字首/尾
5. C+h C+d 删掉光标前后的自负
6. C+y 粘贴至光标后
7. C+r 搜索命令历史，这个较常用

剩下一些不常用的就不介绍了，目前这几个差不多已经够用了，等什么时候去官方文档上看看发现更好的再来补充几个

关于备份，配置文件位于

```
~/Library/Preferences/com.googlecode.iterm2.plist
```

可以把这个文件备份下来，等下次换环境了直接导入也免得重新配置