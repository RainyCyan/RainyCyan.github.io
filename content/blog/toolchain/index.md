---
title: "工具链"
date: 2026-06-23T00:00:00+08:00
tags: [工具链]
series: []
featured: false
description: "如果你想系统了解开发的常用Tools包，请系统学习missing-semester,这是你最不会后悔的一堂课程"
draft: true
ShowToc: true
TocOpen: true
---

## 工具清单
### SHELL
> TODO: short intro, 需要和现在的cli关联
与可视化界面相比，shell是更原生的计算机交互接口，这也是现如今的agent为什么通通基于shell去做TUI的原因。

事实上存在很多shell,比如windows的cmd,powershell,linux的bash,zsh等。我们不会讲解windows上的powershell,事实上，powershell做的是一坨shit,这不是比喻。


### 基础工具

#### Linux常用工具/命令合集

> note: 这里作为收录的临时集中地，如果有比较大块的内容，会独立出去作为单独小结

- **ls**
- **top/htop**
- **curl/wget**
- **ssh/scp/rsync**
- **df/du**
- **mount/umount**
- **tar/gzip/zip**
- **cat/less/head/tail**
- **awk**
- **sort/uniq/wc**
- **chmod**
- **alias**
- **man/info/tldr**
- **history**
- **grep**
- **sed**
#### git -- version control, code history, long long long live
keyword:vcs(version control system)记录版本变动信息；快照，Git 将顶级目录中的文件和文件夹作为集合，并通过一系列快照来管理其历史记录。
> TODO:补充CR,PR/MR以及amend,rebase,squash的协同工作

- Git的数据模型
有向无环图DAG

#### welcome `vim` hotel
> vim不是一种模式，vim是一种处境<狗头>

首先,我们把编程这个任务拆分一下:理解/阅读(read)约占 50%–58%,是最大的一块;导航/定位(navigate)约占
30%–35%;编辑(edit/write)约占 5%–20%,其中"从零敲入新代码"只有约 5%,是最小的一块。

因此,根据第一性原理,最该优先优化的是 read 和 navigate。我们从这两者中抽出一个最核心的动作——移动
(move)。又因为这是在编辑场景下,我们不希望手离开键盘去够鼠标,所以移动只能靠按键完成。而定位的关键之
一是按关键词跳转,这个交给命令来做。至于占比最小、只做追加式写入的 write,设一道门槛即可——用 i 显式
进入这个模式。

vim 把按键动作次数饭比喻行为频率分配，划分了不同的“模式”：

1. Normal Mode普通模式——对应read + navigate的主场
这是vim的默认状态，也是整个设计的地基，vim的每个按键在这里默认都是命令，而不是“插入字符”。
- 全部的导航动作：w b e /f t/


2. Insert Mode插入模式——多一次按键
对应频率最低的

vim 的哲学在于它成功的解耦了程序员的代码编辑任务，编程时，你的大部分时间都花在代码间导航、阅读代码片段和修改代码上，而不是长时间连续输入，或者从头到尾线性地通读文件。
移动：

hjkl(左下上右)
w(ord),b(egin),e(nd)
0(start pos of line),^(non-empty character)


#### tmux

tmux 是一个终端多路复用器，用于在单个终端窗口中管理多个会话。即使断开连接，会话也不会丢失。

##### 安装

```sh
# macOS
brew install tmux

# Linux
sudo apt install -y tmux
```

##### 配置

配置文件为 `~/.tmux.conf`：

```bash
# 启用鼠标支持（滚轮滚动、选择面板）
set -g mouse on

# 增大滚动缓冲区
set -g history-limit 50000
```

##### 常用命令

```bash
tmux new -s my_session       # 新建会话
tmux attach -t my_session    # 连接会话
tmux ls                      # 列出会话
tmux kill-session -t my_session  # 删除会话
```

##### 常用快捷键

> 默认 prefix 为 `Ctrl+b`，先按 prefix 再按对应键。

| 功能 | 快捷键 |
| --- | --- |
| 分离会话 | `d` |
| 列出会话 | `s` |
| 新建窗口 | `c` |
| 切换窗口 | `n` / `p` / `0-9` |
| 水平分屏 | `"` |
| 垂直分屏 | `%` |
| 切换面板 | 方向键 |
| 进入复制模式（滚动） | `[`，`q` 退出 |
