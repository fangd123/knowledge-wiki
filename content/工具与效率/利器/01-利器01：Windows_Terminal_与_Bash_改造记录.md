---
title: "利器01：Windows Terminal 与 Zsh 改造记录"
date: 2026-04-07
author: fangd123
license: CC BY-NC-SA 4.0
category: 工具与效率
series: 利器
series_order: 1
tags:
  - 工具与效率
  - 利器
ai_assisted: true
status: published
---

# 利器01：Windows Terminal 与 Zsh 改造记录

我一般在 Windows 上开终端，通过 SSH 连到 Ubuntu 工作。新环境里我想先办妥三件小事：字看得清，常用命令不用从头敲，合上电脑以后还能接着做。它们看起来都发生在一个黑窗口里，要改的却是三个不同地方。

## 窗口：字体装在看屏幕的这台电脑上

我用 Windows Terminal。它可以从微软应用商店安装。打开设置，选中**实际用来连 SSH 的配置文件**，在“外观”里换字体；不用为了改字号就从头抄一份 `settings.json`。

我主要修改了字体和字号：大屏幕上用 14，字体喜欢 Fira Code。如果提示符带图标，就从 [Nerd Fonts 官方发布页](https://github.com/ryanoasis/nerd-fonts/releases) 找 FiraCode 的补字形版本，装在 Windows 上，再到 Terminal 里选它。远端换多少次字体，也救不了本地窗口里那个空心方框。

## 输入：在远端配置 Zsh

SSH 连上 Ubuntu 以后，再在**远端**安装工具：

```sh
sudo apt install zsh git tmux
zsh
```

第二行是进入 Zsh，不是把所有用户的默认 Shell 换掉。我习惯用 [Oh My Zsh](https://ohmyz.sh/) 管插件；如果要跟着装，从项目首页查看安装说明。已有 `~/.zshrc` 的话先备份，安装器的改动看清再接受。直接用 Zsh 也可以，不装框架不会影响后面的 tmux。

我常用两个插件：[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 会参考历史命令给出灰色建议；[fast-syntax-highlighting](https://github.com/zdharma-continuum/fast-syntax-highlighting) 在输入时着色。前者帮我少打几个字，后者让我在按回车以前多看一眼，但颜色正常不保证命令正确。

用 Oh My Zsh 时，按各自项目的安装说明把插件放进自定义插件目录，再在 `~/.zshrc` **已有的** `plugins=(...)` 里加名字。例如：

```zsh
plugins=(git zsh-autosuggestions fast-syntax-highlighting)
```

如果还想在提示符上看 Git 分支，可以按 [Starship 官方指南](https://starship.rs/guide/) 安装，再在 `~/.zshrc` 末尾加一行 `eval "$(starship init zsh)"`。改完在 Zsh 里运行 `source ~/.zshrc`。不要往 `.bashrc` 里加这一行，也不必把同一行反复追加。

## 会话：断开 SSH 以后怎么接回来

这是我折腾终端最值的一步。先在服务器上开一个有名字的会话：

```sh
tmux new -s work
```

在里面照常干活。要离开时，按默认前缀 `Ctrl-b`，松开，再按 `d`。下次 SSH 连回来，先用 `tmux ls` 看看，再用 `tmux attach -t work` 回到刚才的窗口。留下来的是**服务器上仍在运行的会话**；服务器重启、任务自己退出，都不是重新连接能挽回的。

我后来用了 [Oh My Tmux](https://github.com/gpakosz/.tmux)。已有 tmux 配置不必覆盖；先比对项目说明和自己的键表。下面四个是我当时这套配置里用得最多的，不是原版 tmux 的通用键位：

| 按什么 | 我用它做什么 |
|---|---|
| `Ctrl-a` | 当作 `Ctrl-b` 之外的第二个前缀，手更顺。 |
| 前缀，再按 `m` | 切换鼠标模式。要把文字复制到 Windows 时，还要区分选区归终端还是归 tmux。 |
| 前缀，再按 `-` | 把窗格分成上下两块，下面一块看日志。 |
| 前缀，再按 `_` | 把窗格分成左右两块，并排看命令和结果。 |

不确定自己的配置绑定了什么，运行 `tmux list-keys` 比多按几次可靠。刚接触 tmux 可以先读[阮一峰的入门教程](https://www.ruanyifeng.com/blog/2019/10/tmux.html)。已经装完、只剩下某个地方不对劲，可以直接看[终端环境改造：按症状排查](../终端环境改造.md)。
