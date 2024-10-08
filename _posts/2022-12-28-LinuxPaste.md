---
layout: article
title: Linux的各种复制粘贴
date: 2022-12-12
key: P2022-12-12
tags: ["Linux"]
show_author_profile: false
comment: true
sharing: true
aside:
  toc: true
---

Linux的各种复制粘贴。VIM、tmux、nvim、和终端之间的复制粘贴。

更新：已经换为在tumx中用Neovim了。

<!--more-->

环境为 Ubuntu 22.04 LTS

# Vim

vim的剪贴板功能是寄存器(register)，和系统剪贴板(system clipboard)不是一个东西。

vim如果复制的时候有行号，不想要行号显示可以使用`:set nonu` 关闭行号，使用`:set nu` 恢复行号。

## 确认版本

根据平台不同，要分两种情况。

```bash
which vim
```

可以看到当前默认使用的vim是哪个。用下面命令确定你属于哪一种情况，

```bash
vim --version | grep clipboard
```

`vim --version`可以看到当前使用的vim支持哪些feature，'+'前缀表示拥有的feature，'-'前缀表示未拥有。

如果找到的是负号开头的`-clipboard`，说明你的Vim不支持系统剪切板，需要先重新安装vim。如果结果里你找到加号开头的`+clipboard`，那么则没问题跳过重新安装那一步继续即可。

## 重新安装

`-clipboard`的情况，想要Vim跟系统剪切板交互则需要重新安装合适的版本。

Vim有很多版本。如果安装时直接用了vim名称

```bash
sudo apt install vim
```

那么安装的会是 `vim-common` 套件, 这个套件只会安装文字模式的 vim, 不会安装图形化版本的 Gvim。没有图形化版本的套件, 也不会加上 clipboard/xterm_clipboard 模组, 也就是无法让 vim 复制资料到系统剪贴板, 也无法从系统剪切板贴资料到 vim 中。

`vim-gui-common`是通用的图形化版本。`vim-gtk/vim-gtk3`是搭配gtk图形界面的软件版本。其中`vim-gtk`是GTK2版本。

一般情况下用 `vim-gtk3`即可。

```bash
apt list --installed | grep vim
sudo apt remove vim
sudo apt autoremove
apt search vim-gtk3
apt show vim-gtk3
sudo apt update
sudo apt install vim-gtk3
```

## .vimrc

把Vim默认无名寄存器`""` 和系统剪贴板关联

```bash
set clipboard=unnamed
```

在可视模式(-- VISUAL --)下，按Y键将当前内容复制到系统剪切板

```bash
vnoremap Y "+y
```

## Neovim

说实话被vim剪贴板的事弄得一直不太满意。有点不想折腾了，直接装 Neovim 吧

添加 PPA(Personal Package Archive) 存储库并安装：

```bash
sudo add-apt-repository ppa:neovim-ppa/stable
sudo apt update

sudo apt install neovim
```

PPA是软件包来自于个人或团队提供的存储库。测试安装：`nvim --version`

从 Neovim 0.10.0 ([#25872](https://github.com/neovim/neovim/pull/25872)) 开始，Neovim 已经内置了原生的 [ANSI OSC52](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html#h3-Operating-System-Commands) 支持，能将文本复制到系统剪贴板。

完成安装后执行`nvim`进入Neovim，再输入：

```
:checkhealth
```

根据提示信息，检查是否有内容需要修改，也要检查`Clipboard (optional)`部分。

最后，还需要额外把 `sudoedit` 默认的编辑器从vim换到neovim

```bash
sudo update-alternatives --config editor
```

该命令会列出系统中可用的编辑器选项，并要求选择一个作为默认的编辑器。

# tmux

在 .tmux.conf 中添加配置：

```
set -s set-clipboard on
```

tmux内部复制/粘贴文本的通用方式：

(1) 按下`Ctrl + a`后松开手指，然后按`[`

(2) 用鼠标左键长按选中文本，被选中的文本会被自动复制到tmux的剪贴板

(3) 按下`Ctrl + a`后松开手指，然后按`]`，会将剪贴板中的内容粘贴到光标处。

另外，复制整个文件的文本，可以退出tmux来复制，用Linux终端下的方式复制。

退出 tmux

cat filename

# 终端

Linux终端下的复制方式：

(a)左键长按选中要复制的内容

 然后按ctrl+Ins复制 shift+Ins粘贴

(b)shift+鼠标左键长按选中

 然后按ctrl+Ins复制shift+Ins粘贴

(c)鼠标左键选中开头的若干字符

 按住Shift，同时鼠标左击要复制内容的末尾，此时会选中从开头到左键点击末尾位置的所有内容

 然后按ctrl+Ins复制shift+Ins粘贴

参考链接:

> [Vim 剪贴板里面的东西 粘贴到系统粘贴板](https://www.zhihu.com/question/19863631/answer/442180294)
