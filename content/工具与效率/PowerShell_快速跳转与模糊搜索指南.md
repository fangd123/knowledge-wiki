---
title: "PowerShell 快速跳转与模糊搜索指南"
date: 2026-05-06
author: fangd123
license: CC BY-NC-SA 4.0
category: 工具与效率
tags:
  - 工具与效率
ai_assisted: true
status: published
---

# PowerShell 快速跳转与模糊搜索指南

这份文档记录了当前这台 Windows / PowerShell 环境里已经配置好的几组工具：

- `zoxide`
- `z` / `zi` / `fz`
- `fzf`
- `PSFzf`
- `fd`
- `bat`

目标很简单：

- 更快跳目录
- 更快找文件
- 更顺手地复用命令历史
- 少敲完整路径

## 一览

已经配置好的核心能力：

- `z foo`：按历史权重跳到最可能的目录
- `zi foo`：交互式选择目录后跳转
- `fz foo`：`zi` 的短别名
- `Ctrl+t`：模糊选择文件或路径，插入到当前命令行
- `Alt+c`：模糊选择目录，然后直接 `cd` 进去
- `Ctrl+r`：模糊搜索命令历史
- `Tab`：模糊补全
- `fd`：高速搜文件/目录
- `bat`：带高亮的文件预览

## 已安装版本

```powershell
zoxide 0.9.9
fzf 0.72.0
PSFzf 2.7.10
fd 10.4.2
bat 0.26.1
```

## 1. zoxide

`zoxide` 可以理解成“会学习你跳转习惯的 `cd`”。

它会记住你经常进入的目录，然后根据关键字和历史权重给出最可能的目标。

### 常见命令

```powershell
z foo
z foo bar
z langgraph
z xiaojin
```

含义：

- `z foo`：跳到最匹配 `foo` 的目录
- `z foo bar`：同时按多个关键词匹配
- `z langgraph`：适合项目名明确的时候
- `z xiaojin`：适合一组相关项目的总入口

### 常见辅助命令

```powershell
zoxide query xiaojin
zoxide query -l xiaojin
zoxide query -l -s xiaojin
zoxide add D:\projects\company\xiaojin_langgraph
zoxide edit
```

含义：

- `zoxide query xiaojin`：看默认会跳到哪里
- `zoxide query -l xiaojin`：列出所有匹配候选
- `zoxide query -l -s xiaojin`：连分数一起列出来
- `zoxide add ...`：手动把目录加入数据库，或提高它的权重
- `zoxide edit`：手动编辑数据库

### 特色

- 越常进的目录，命中越靠前
- 支持多关键词
- 很适合项目目录、工作区目录、文档目录
- 比纯模糊搜索更“懂你”

## 2. z / zi / fz

这是你平时最常用的入口。

### `z`

```powershell
z xiaojin
z langgraph
z deploy
```

特点：

- 优先走 `zoxide` 的历史数据库
- 适合“我知道大概叫啥，而且我平时经常进”
- 当前环境里已经做了增强：
  如果 `zoxide` 数据库没有命中，会退化成目录搜索，而不是直接失败

### `zi`

```powershell
zi xiaojin
zi langgraph
zi report
```

特点：

- 交互式选择
- 适合“候选太多，我想看一眼再选”
- 背后会调用 `fzf`

### `fz`

```powershell
fz xiaojin
fz langgraph
```

特点：

- `fz` 是我加的短别名
- 等价于 `zi`
- 更省手

### 当前机器上的定制

为了更符合你当前的使用习惯，我已经做了这些处理：

- 预热了一批 `xiaojin` 相关项目目录到 `zoxide`
- 把 `D:\projects\company\xiaojin_langgraph` 的权重调高了

所以当前：

```powershell
z xiaojin
z langgraph
```

默认都会跳到：

```powershell
D:\projects\company\xiaojin_langgraph
```

## 3. fzf

`fzf` 是通用的模糊搜索器，本身不负责跳目录，但很多工具都拿它做交互层。

### 最简单的用法

```powershell
fd xiaojin | fzf
Get-ChildItem -Recurse | Select-Object -ExpandProperty FullName | fzf
```

特点：

- 输入几个字母就能筛选
- 支持键盘交互
- 适合做“人眼确认后再选”

### 当前机器上的体验增强

已经配置好了：

- 反向布局
- 边框
- 右侧预览窗口
- 目录和文件都能预览

预览逻辑：

- 目录：列出目录内容
- 文件：优先用 `bat` 高亮预览

## 4. PSFzf

`PSFzf` 是 `fzf` 在 PowerShell 里的整合层。

你最常用的是快捷键。

### 快捷键

```text
Ctrl+t   模糊选择文件或路径，并插入到当前命令行
Alt+c    模糊选择目录，并直接跳进去
Ctrl+r   模糊搜索历史命令
Tab      模糊补全
```

### 使用示例

#### `Ctrl+t`

先输入：

```powershell
code 
```

然后按 `Ctrl+t`，选一个文件，效果类似：

```powershell
code D:\projects\company\xiaojin_langgraph\README.md
```

#### `Alt+c`

不用先敲 `cd`，直接按 `Alt+c`，选目录后会自动进入。

#### `Ctrl+r`

适合找以前跑过但忘了完整内容的命令，比如：

- `docker compose up`
- `npm run dev`
- `uv run ...`

#### `Tab`

在路径、文件名、部分命令参数补全时，会弹出模糊选择界面。

## 5. fd

`fd` 是比 `find` / `Get-ChildItem -Recurse` 更快、更顺手的搜索工具。

### 常见用法

```powershell
fd xiaojin
fd langgraph D:\projects\company
fd -t d xiaojin
fd -t f README
fd -e md guide
```

含义：

- `fd xiaojin`：搜名字里带 `xiaojin` 的文件和目录
- `fd langgraph D:\projects\company`：只在指定目录里搜
- `fd -t d xiaojin`：只搜目录
- `fd -t f README`：只搜文件
- `fd -e md guide`：只搜扩展名为 `.md` 的文件

### 特色

- 很快
- 参数短
- 默认体验友好
- 很适合跟 `fzf` 配合

### 配合 `fzf`

```powershell
fd -t d xiaojin | fzf
fd -e md . | fzf
```

## 6. bat

`bat` 是 `cat` 的增强版，带语法高亮、行号、主题。

### 常见用法

```powershell
bat README.md
bat .\app.py
bat --style=plain .\notes.txt
bat --line-range=:200 .\large.log
```

### 特色

- 看代码、配置、日志更舒服
- 非常适合做 `fzf` 的预览器
- 比 `Get-Content` 更适合“快速扫一眼”

## 7. 典型工作流

### 工作流 1：直接跳主项目

```powershell
z xiaojin
```

适合：

- 主项目很固定
- 你不想每次手选

### 工作流 2：一组项目里手动挑

```powershell
fz xiaojin
```

适合：

- `xiaojin` 下面有很多仓库
- 今天到底进哪个分支目录还不确定

### 工作流 3：打开某个文件

```powershell
code 
```

然后按 `Ctrl+t`。

适合：

- 路径很深
- 只记得文件名的一部分

### 工作流 4：回忆历史命令

按 `Ctrl+r`，输入关键字，例如：

```text
docker
uv
pytest
```

适合：

- 忘了完整命令
- 只记得一小段

### 工作流 5：搜目录后直接进入

按 `Alt+c`，输入：

```text
langgraph
```

然后选目录。

适合：

- 不一定依赖 `zoxide` 历史
- 更想“现场搜一遍再跳”

## 8. 什么时候用哪个

### 用 `z`

当你：

- 经常去同一个项目
- 希望一步直达
- 接受结果受历史权重影响

### 用 `zi` / `fz`

当你：

- 知道关键词
- 但候选很多
- 想看一眼再决定

### 用 `Alt+c`

当你：

- 单纯想搜目录
- 不在乎是否来自 `zoxide` 历史

### 用 `Ctrl+t`

当你：

- 已经在输入命令
- 只是还差一个路径或文件名

### 用 `fd`

当你：

- 想快速搜文件或目录
- 想把结果再接给 `fzf`

## 9. 常见排查

### `z foo` 没跳到你想要的目录

先看当前默认命中：

```powershell
zoxide query foo
zoxide query -l -s foo
```

如果要提高某个目录权重：

```powershell
zoxide add D:\projects\company\xiaojin_langgraph
```

### 候选太多

改用：

```powershell
zi foo
fz foo
```

### 想看所有候选

```powershell
zoxide query -l foo
```

### 想搜但不依赖历史

```powershell
fd -t d foo | fzf
```

## 10. 推荐记忆顺序

先熟悉这几个就够用了：

1. `z foo`
2. `fz foo`
3. `Ctrl+t`
4. `Alt+c`
5. `Ctrl+r`
6. `fd pattern`

如果你把这 6 个用顺了，PowerShell 下的路径操作效率就会明显上来。
