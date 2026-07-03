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

#### vim

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
