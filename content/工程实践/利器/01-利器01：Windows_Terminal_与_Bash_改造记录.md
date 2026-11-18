---
title: "利器01：Windows Terminal 与 Bash 改造记录"
date: 2026-04-07
author: fangd123
license: CC BY-NC-SA 4.0
category: 工程实践
series: 利器
series_order: 1
tags:
  - 工程实践
  - 利器
ai_assisted: true
status: review
---

> 由于微信公众号无法直接跳转外链，大家可以点击**阅读原文**进入我的博客浏览

*“利器”系列说明：“工欲善其事必先利其器”，有了称手好用的工具，才能最大程度发挥出自己的能力，提高生产力和效率，避免无效加班。因此，“利器”系列将会记录我对于各种工具的改造，以及我是如何组合应用它们发挥价值。*

---

这篇文章主要记录我 Terminal & Bash 改造，Terminal & Bash 是所有程序员在新的开发环境下第一个接触也是最常接触的地方，因此，这里的作为“利器”系列的第一篇，分享给大家。

**注：以下内容中，涉及到下载的链接，均已经过国内网络优化地址，方便大家快速下载**

## Terminal ：Windows Terminal

Windows环境下，我选择 Windows Terminal

**选择原因**：其他的 Terminal 存在一个不好解决的问题，就是在使用远程Tmux的情况下，无法使用鼠标选择复制粘贴，同时Windows Terminal 也拥有不输其他 Terminal 个性化配置功能

**安装方法**：直接在微软应用商店搜索“Windows Terminal”安装即可。

### 配置

有两种配置方式，一种是交互式，另一种是编辑配置文件setttings.json，建议初次配置使用交互式的配置方式即可。配置完成后，可以将配置文件导出备份到云盘上，方便以后直接使用。

我主要修改了字体和字号，字号设置为14，方便大屏幕阅读，字体用的是“FiraCode”，这是一种针对于编程的字体，详细的介绍可以看这里：[FiraCode](https://github.com/ryanoasis/nerd-fonts/tree/master/patched-fonts/FiraCode)

字体的官方下载链接：[FiraCode.zip](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/FiraCode.zip)

阿里云下载链接：

> 「firacode」，点击链接保存，或者复制本段内容，打开「阿里云盘」APP ，无需下载极速在线查看，视频原画倍速播放。 链接：https://www.aliyundrive.com/s/Wx5ZFrUhquY
> 

---

## Shell 配置

因为我一般都是在本地通过SSH远程连接到服务器Ubuntu系统上，因此，这里只讨论Ubuntu环境下的Shell配置。

### 安装 ZSH

在Ubuntu系统下，执行

```bash
sudo apt install -y zsh curl wget git tmux
```

### 安装 Oh My Zsh

Oh My Zsh 是 Zsh 的配置管理器，包括插件，主题等的配置，并且已经默认了许多实用功能，集成了相当多的插件（大部分默认不启用）有兴趣的朋友可以进入它的官网获取更多信息：

[Oh My Zsh - a delightful & open source framework for Zsh](https://ohmyz.sh/)

想快速了解的朋友，可以看看它的Cheatsheet：

[https://github.com/ohmyzsh/ohmyzsh/wiki/Cheatsheet](https://github.com/ohmyzsh/ohmyzsh/wiki/Cheatsheet)

安装过程

```bash
cd ~
wget https://pd.zwc365.com/https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
# 设置远端地址为github镜像地址
Export REMOTE=https://hub.fastgit.org/ohmyzsh/ohmyzsh.git
sh install.sh
```

### 安装 starship

```bash
sh -c "$(curl -fsSL https://pd.zwc365.com/https://starship.rs/install.sh)" -- -B https://hub.fastgit.org/starship/starship/releases
echo 'eval "$(starship init zsh)"' >> ~/.zshrc
```

### 安装 Shell 语法高亮插件

插件地址：

[Fast Syntax Highlighting (F-Sy-H)](https://github.com/zdharma-continuum/fast-syntax-highlighting)

效果预览：

> 图注：https://pic1.zhimg.com/v2-f5e23c855f18f66809237e5bf31ec798_b.png

安装步骤：

1. 执行命令：

```bash
git clone https://hub.fastgit.org/zdharma-continuum/fast-syntax-highlighting.git \
  ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/fast-syntax-highlighting
```

2. 打开`~/.zshrc` 文件，找到 `plugins`，添加`fast-syntax-highlighting`插件

```bash
 plugins=(git fast-syntax-highlighting)
```

3. 激活ZSH配置，执行命令

```
source ~/.zshr
```

### 安装 Shell 自动完成插件

效果预览：

[https://asciinema.org/a/37390](https://asciinema.org/a/37390)

1. 下载插件

```bash
git clone https://hub.fastgit.org/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

2. 添加插件

打开`~/.zshrc` 文件，找到 `plugins`

```bash
 plugins=(git fast-syntax-highlighting zsh-autosuggestions)
```

3. 激活插件

```bash
source ~/.zshrc
```

---

## Tmux

Tmux 是一款强大的终端复用器（这个翻译有点废），通俗一点的说，就是在关闭SSH远程连接的情况下，终端里运行的程序依然不会断开。

Tmux 的基本介绍和使用，可以参考阮一峰的博客，链接在这里：

[Tmux 使用教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2019/10/tmux.html)

### Tmux 配置文件 Oh My Tmux

效果预览：

> 图注：https://pic3.zhimg.com/v2-a373906b98b5ed7cfadb506017c27eea_b.png

安装命令如下：

```bash
cd
git clone https://hub.fastgit.org/gpakosz/.tmux.git
ln -s -f .tmux/.tmux.conf
cp .tmux/.tmux.conf.local .
```

安装完成后，进入Tmux便可以看到新的界面了

我用的比较多的快捷键：

1. 添加了`C-a`作为C-b的第二选择（从键位的角度来说，`C-a` 更方便一些）
2. `<prefix> m` 绑定鼠标模式
3. `<prefix> -` 纵向分割当前面板
4. `<prefix> _` 横向分割当前面板

更加详细的介绍，可以进入作者的github页面了解，链接如下：

[https://github.com/gpakosz/.tmux](https://github.com/gpakosz/.tmux)

---

以上便是Terminal和Shell的改造全过程，希望这篇文章能够帮助你在终端界面中提高效率。

当然，这并不是改造的结束，相关的工具和插件其实是非常庞杂的，也有许多值得进一步探索的地方，我也只是用了这些工具的冰山一角而已。

我会在今后的文章中，逐渐加入一些针对性的环境配置和工具使用教程。

如果大家想推荐自己使用的工具，欢迎再评论区留言，我会即使将推荐内容更新至文章当中。

---

题图 Photo by [Andrew Svk](https://unsplash.com/@andrew_svk?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/t/travel?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

参考文章：[HowTo: My Terminal & Shell setup - Hyper.js + ZSH + starship ☄️🔥](https://tjay.dev/howto-my-terminal-shell-setup-hyper-js-zsh-starship/)

