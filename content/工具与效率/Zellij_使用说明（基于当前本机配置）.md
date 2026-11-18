---
title: "Zellij 使用说明（基于当前本机配置）"
date: 2026-05-06
author: fangd123
license: CC BY-NC-SA 4.0
category: 工具与效率
tags:
  - 工具与效率
ai_assisted: true
status: published
---

# Zellij 使用说明（基于当前本机配置）

这是一张从 tmux 迁过来后留下的键位表，记的是我使用 Zellij 0.44.2 时的配置。**不是原厂速查表**：`Ctrl-a` 进 Tmux 模式、配套的二级键、`ohmytmux` 主题和布局，都是下表成立的前提。原样安装 Zellij 不会得到这套界面；拿不准时，以自己窗口底部的模式提示为准。

如果只是想试这个手感，不用复制我整套配置。先执行 `zellij -s practice` 开个练习会话，看看当前提示栏怎么分屏和 detach，再用 `zellij a practice` 接回。能完成这两个动作，就可以往下挑自己要的键了。

## 当前配置概览

- 主前缀键：`Ctrl-a`
- 兼容前缀键：`Ctrl-b`
- 默认 Shell：`pwsh`
- 默认布局：`ohmytmux`
- 界面布局：主内容区 + 底部 tab 栏 + 最底部按键提示栏
- 主题：`ohmytmux`
- Pane 边框：关闭
- 会话序列化：开启
- Pane 视口序列化：开启

这意味着：

- 你可以继续按接近 `tmux / oh-my-tmux` 的方式来用它。
- 按下前缀键后，底部提示栏会显示当前模式下可用的操作。
- 要暂时离开，选 **detach**；不要把退出最后一个 pane 或终止会话当成 detach。

## 最常用的键位

这部分优先覆盖平时最常用的操作。

| 按键 | 功能 |
| --- | --- |
| `Ctrl-a` | 进入 `Tmux` 模式 |
| `Ctrl-b` | 也可进入 `Tmux` 模式 |
| `Ctrl-a Ctrl-a` | 向程序发送原始 `Ctrl-a` |
| `Ctrl-a Ctrl-b` | 向程序发送原始 `Ctrl-b` |
| `Ctrl-a "` | 上下分屏 |
| `Ctrl-a %` | 左右分屏 |
| `Ctrl-a h/j/k/l` | 在 pane 之间移动焦点 |
| `Ctrl-a c` | 新建 tab |
| `Ctrl-a 1..9` | 跳转到指定 tab |
| `Ctrl-a n / p` | 切到下一个 / 上一个 tab |
| `Ctrl-a ,` | 重命名当前 tab |
| `Ctrl-a x` | 关闭当前 pane |
| `Ctrl-a z` | 当前 pane 全屏 / 取消全屏 |
| `Ctrl-a [` | 进入滚动 / 复制模式 |
| `Ctrl-a d` | detach 当前 session |
| `Ctrl-a w` | 打开 session manager |
| `Ctrl-a r` | 打开 configuration 面板 |
| `Ctrl-a Space` | 切换布局变体 |

## 模式切换

Zellij 是一个带“模式”的终端复用器。你进入不同模式后，同一组按键的含义会变化。

| 按键 | 功能 |
| --- | --- |
| `Ctrl-a` | 进入 `Tmux` 模式 |
| `Ctrl-b` | 进入 `Tmux` 模式（兼容保留） |
| `Ctrl-p` | 进入 `Pane` 模式 |
| `Ctrl-t` | 进入 `Tab` 模式 |
| `Ctrl-n` | 进入 `Resize` 模式 |
| `Ctrl-h` | 进入 `Move` 模式 |
| `Ctrl-s` | 进入 `Scroll` 模式 |
| `Ctrl-o` | 进入 `Session` 模式 |
| `Ctrl-g` | 锁定 / 解锁 |
| `Esc` / `Enter` | 从大多数模式回到 `Normal` 模式 |
| `Ctrl-q` | 退出 Zellij |

说明：

- 日常使用里，你可以主要把它当成“`Ctrl-a` 前缀模式”来理解。
- 其余模式更适合做批量操作、调整尺寸、管理会话。

## Tmux 模式

这是最像 `oh-my-tmux` 的一层，也是最推荐优先记住的一层。

| 按键 | 功能 |
| --- | --- |
| `[` | 进入滚动 / 搜索模式 |
| `Ctrl-a` | 发送原始 `Ctrl-a` |
| `Ctrl-b` | 发送原始 `Ctrl-b` |
| `"` | 上下分屏 |
| `%` | 左右分屏 |
| `z` | pane 全屏切换 |
| `c` | 新建 tab |
| `,` | 重命名 tab |
| `p` | 上一个 tab |
| `n` | 下一个 tab |
| `1..9` | 跳到指定 tab |
| `h/j/k/l` | 切换焦点到左 / 下 / 上 / 右 pane |
| 方向键 | 同上，用方向键切换 pane |
| `o` | 切到下一个 pane |
| `d` | detach |
| `Space` | 切换布局 |
| `x` | 关闭当前 pane |
| `w` | 打开 session manager |
| `r` | 打开 configuration |

## Pane 模式

进入方式：`Ctrl-p`

| 按键 | 功能 |
| --- | --- |
| `h/j/k/l` 或方向键 | 在 pane 之间移动焦点 |
| `p` | 在最近两个 pane 之间切换 |
| `n` | 新建 pane |
| `d` | 在下方新建 pane |
| `r` | 在右侧新建 pane |
| `s` | 新建 stacked pane |
| `x` | 关闭当前 pane |
| `f` | 全屏切换 |
| `z` | 切换 pane frame 显示 |
| `w` | 显示 / 隐藏 floating panes |
| `e` | 嵌入 / 浮动切换 |
| `c` | 重命名当前 pane |
| `i` | 固定 / 取消固定 pane |
| `Ctrl-p` | 返回 `Normal` |

## Tab 模式

进入方式：`Ctrl-t`

| 按键 | 功能 |
| --- | --- |
| `h/k` 或左 / 上 | 上一个 tab |
| `l/j` 或右 / 下 | 下一个 tab |
| `n` | 新建 tab |
| `x` | 关闭当前 tab |
| `r` | 重命名当前 tab |
| `s` | 切换同步输入到当前 tab 的所有 pane |
| `b` | 把当前 pane 拆成独立 tab |
| `[` | 把 pane break 到左边 |
| `]` | 把 pane break 到右边 |
| `1..9` | 跳转到指定 tab |
| `Tab` | 在最近两个 tab 之间切换 |
| `Ctrl-t` | 返回 `Normal` |

## Resize 模式

进入方式：`Ctrl-n`

| 按键 | 功能 |
| --- | --- |
| `h/j/k/l` 或方向键 | 沿指定方向增大当前 pane |
| `H/J/K/L` | 沿指定方向减小当前 pane |
| `+` / `=` | 整体增大 |
| `-` | 整体减小 |
| `Ctrl-n` | 返回 `Normal` |

## Move 模式

进入方式：`Ctrl-h`

| 按键 | 功能 |
| --- | --- |
| `h/j/k/l` 或方向键 | 把当前 pane 移到对应方向 |
| `n` / `Tab` | 轮换 pane 位置 |
| `p` | 反向轮换 pane 位置 |
| `Ctrl-h` | 返回 `Normal` |

## Scroll / Search 模式

进入方式：`Ctrl-s`，或者 `Ctrl-a [`

### Scroll 模式

| 按键 | 功能 |
| --- | --- |
| `j/k` 或下 / 上 | 向下 / 向上滚动 |
| `Ctrl-f` / `l` / `PageDown` | 向下翻页 |
| `Ctrl-b` / `h` / `PageUp` | 向上翻页 |
| `d` | 向下半页 |
| `u` | 向上半页 |
| `s` | 进入搜索输入 |
| `e` | 用编辑器打开 scrollback |
| `Ctrl-c` | 回到底部并退出到 `Normal` |

### Search 模式

| 按键 | 功能 |
| --- | --- |
| `n` | 搜索下一个结果 |
| `p` | 搜索上一个结果 |
| `c` | 切换大小写敏感 |
| `w` | 切换结果循环 |
| `o` | 切换整词匹配 |
| `Ctrl-c` | 回到底部并退出到 `Normal` |

### EnterSearch 模式

| 按键 | 功能 |
| --- | --- |
| `Enter` | 提交搜索词并进入 `Search` |
| `Esc` / `Ctrl-c` | 回到 `Scroll` |

## Session 模式

进入方式：`Ctrl-o`

| 按键 | 功能 |
| --- | --- |
| `d` | detach 当前 session |
| `w` | 打开 session manager |
| `c` | 打开 configuration 面板 |
| `p` | 打开 plugin manager |
| `a` | 打开 about |
| `s` | 打开 share 面板 |
| `l` | 打开 layout manager |
| `Ctrl-s` | 切到滚动模式 |
| `Ctrl-o` | 返回 `Normal` |

## 全局快捷键

这些键不需要先进入 `Tmux` 模式，平时就可以直接用。

| 按键 | 功能 |
| --- | --- |
| `Alt-n` | 新建 pane |
| `Alt-f` | 显示 / 隐藏 floating panes |
| `Alt-h/j/k/l` | 在 pane / tab 间快速切换 |
| `Alt-Left/Right/Up/Down` | 同上，方向键版本 |
| `Alt-i` | tab 左移 |
| `Alt-o` | tab 右移 |
| `Alt-=` / `Alt-+` | 放大 |
| `Alt--` | 缩小 |
| `Alt-[` | 上一个布局变体 |
| `Alt-]` | 下一个布局变体 |
| `Alt-p` | 切换 pane 分组 |
| `Alt-Shift-p` | 切换 group marking |

## Zellij 的特色功能

下面这些是 Zellij 相比传统 `tmux` 体验里比较突出的点。

### 1. 模式化操作和底部按键提示

- Zellij 会在不同模式下给出明确的键位含义。
- 我这套配置启用了底部 `status-bar`，按前缀键后会直接显示可用操作提示。
- 这对刚从 tmux 迁移过来的人很友好，不容易忘键。

### 2. 原生布局系统

- Zellij 不只是“分屏”，它还有 layout 概念。
- 你可以在不同布局之间切换，而不是每次都手动重新排 pane。
- 当前配置里：
  - 默认布局是 `ohmytmux`
  - `Ctrl-a Space` 或 `Alt-[` / `Alt-]` 可以切换布局变体

### 3. 会话序列化和恢复

当前配置已开启：

- `session_serialization true`
- `serialize_pane_viewport true`

要分清三种“回来”：

| 发生了什么 | 能回来什么 |
|---|---|
| 只是 detach 或 SSH 掉线；原主机和会话进程还在 | 用 `zellij ls` 找到名字再 attach。看到的是**同一个活会话**，其中的程序可以一直运行。 |
| 会话已结束或主机重启，仍有序列化记录 | 可以尝试按记录重建 tab、pane、布局及保存的视口。屏幕上能看到旧输出，不等于它对应的进程还活着。 |
| 正在跑的计算随关机而退出 | 终端复用器没有保存进程内存；要从程序自己的存档、任务队列或检查点恢复。重开一条命令，是一次新运行。 |

### 4. Floating panes

- 除了普通嵌入式 pane，Zellij 还支持 floating panes。
- 你可以把某个 pane 浮起来，当作临时工具窗来用。
- 当前常用键：
  - `Alt-f`
  - `Ctrl-p` 后按 `w`
  - `Ctrl-p` 后按 `e` 在浮动 / 嵌入之间切换

### 5. Stacked panes

- 支持 stacked pane，这个在传统 tmux 里不是默认核心体验。
- 当前键位：`Ctrl-p` 后按 `s`
- 适合把多条命令或多个小任务叠在同一区域。

### 6. 内建管理面板

Zellij 自带一些很好用的内建面板，而不是完全依赖外部脚本。

- `session-manager`
- `configuration`
- `plugin-manager`
- `layout-manager`
- `about`
- `share`

我最常用的入口：

- `Ctrl-a w` 打开 session manager
- `Ctrl-a r` 打开 configuration
- `Ctrl-o` 进入 session 模式后再按对应字母打开相关面板

### 7. 强一点的滚动和搜索体验

- 支持滚动模式、搜索模式、半页翻动、整页翻动
- 支持搜索选项切换：大小写、整词、循环
- 支持把 scrollback 直接交给编辑器查看

### 8. Plugin 体系

当前配置已启用这些插件别名：

- `tab-bar`
- `status-bar`
- `compact-bar`
- `session-manager`
- `configuration`
- `plugin-manager`
- `strider`
- `about`

用起来最直观的体现就是：

- tab 栏和状态提示栏本身就是插件渲染出来的
- Zellij 的界面能力比传统“纯终端文本栏”更灵活

## 当前这套配置和 oh-my-tmux 的关系

我这套配置的目标不是“完全复制 tmux”，而是：

- 尽量保留 `oh-my-tmux` 的核心手感
- 保留 `Ctrl-a` 前缀习惯
- 保留底部按键提示
- 利用 Zellij 自己更现代的功能

已经比较接近的部分：

- `Ctrl-a` 前缀
- `"` / `%` 分屏
- `n` / `p` 切 tab
- `1..9` 跳 tab
- `,` 重命名
- `d` detach
- 底部 Powerline 风格栏位

仍然和 tmux 不完全一样的部分：

- Zellij 是模式化的，不是单纯“所有操作都挂在前缀后面”
- 部分能力是通过内建面板完成，而不是 shell 脚本或 tmux 插件
- floating panes、stacked panes、layout manager 这些更像 Zellij 自己的路子

## 命令行常用命令

| 命令 | 功能 |
| --- | --- |
| `zellij` | 启动新 session |
| `zellij ls` | 列出 session |
| `zellij a <session-name>` | attach 到已有 session |
| `zellij -s <name>` | 以指定名字启动 session |
| `zellij kill-session <name>` | 杀掉某个 session |
| `zellij kill-all-sessions` | 杀掉所有 session |

## 推荐记忆顺序

如果你想尽快顺手，建议按这个顺序记：

1. `Ctrl-a`
2. `Ctrl-a "` / `Ctrl-a %`
3. `Ctrl-a h/j/k/l`
4. `Ctrl-a c` / `Ctrl-a n` / `Ctrl-a p`
5. `Ctrl-a [` / `Ctrl-a d`
6. `Ctrl-a w`
7. `Ctrl-p` / `Ctrl-t` / `Ctrl-s`

## 一句话总结

我把 Zellij 调成了“`oh-my-tmux` 手感优先，但保留 Zellij 现代特性”的形态：

- 用起来像 tmux
- 提示比 tmux 友好
- pane / tab / session 管理更现代
- 会话恢复、floating pane、stacked pane、layout manager 是它比传统 tmux 更有意思的地方
