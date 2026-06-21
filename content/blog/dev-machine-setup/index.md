---
title: "开发机常用配置"
date: 2026-06-22T00:00:00+08:00
tags: [开发环境, 配置, Linux]
series: []
featured: true
description: "记录开发机上常用的系统配置、工具安装和环境优化，方便新机器快速上手"
draft: true
ShowToc: true
TocOpen: true
---

## 前言

每次拿到一台新开发机（无论是物理机、云服务器还是 WSL），都需要重复做一些基础配置工作。本文记录我常用的配置清单，作为模板方便快速复用。后续需要把里面的一些核心配置成skills。

## 系统基础配置

### 权限控制
对于一个开发机器的权限控制，最最常见的你可能是一个普通user，或者你拥有机器的admin账号，在不同权限级别下你可以有不同的策略，我的攻略主要针对普通user的情况。

### 安装AI agent/terminal

### 包管理器换源

**Ubuntu / Debian**

```bash
# 备份原配置
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

# 替换为清华源（以 Ubuntu 24.04 为例）
sudo sed -i 's|http://archive.ubuntu.com|https://mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list
sudo sed -i 's|http://security.ubuntu.com|https://mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list

sudo apt update && sudo apt upgrade -y
```

### 基础开发工具

```bash
sudo apt install -y \
  build-essential \
  cmake \
  gdb \
  git \
  curl \
  wget \
  unzip \
  tar \
  htop \
  tmux \
  ripgrep \
  fd-find \
  tree \
  jq \
  net-tools \
  openssh-server
```

## Shell 配置

### Oh My Zsh

```bash
# 安装 zsh
sudo apt install -y zsh

# 安装 Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 安装常用插件
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 在 ~/.zshrc 中启用插件
# plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

### 常用别名

在 `~/.zshrc` 或 `~/.bashrc` 中添加：

```bash
# 常用命令缩写
alias ll='ls -lah'
alias la='ls -A'
alias l='ls -lh'
alias g='git'
alias gc='git clone'
alias gp='git pull'
alias gf='git fetch'
alias lg='lazygit'

# 目录导航
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'

# 系统
alias df='df -h'
alias du='du -sh'
alias free='free -h'
alias psg='ps aux | grep'

# 网络
alias myip='curl ifconfig.me'
alias ports='netstat -tulanp'
```

## Git 配置

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global core.editor "vim"
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global fetch.prune true

# 别名
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all --decorate"
```

### 生成 SSH Key

```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

## 编程语言环境

### Go

```bash
# 方法一：apt 安装（版本可能较旧）
sudo apt install -y golang-go

# 方法二：官方安装（推荐）
wget https://go.dev/dl/go1.23.0.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.23.0.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.zshrc
echo 'export GOPATH=$HOME/go' >> ~/.zshrc
source ~/.zshrc
```

### Python

```bash
# 安装 pip 和 venv
sudo apt install -y python3-pip python3-venv

# 配置 pip 镜像
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 常用工具
pip install --upgrade pip
pip install ipython jupyter black flake8 pytest
```

### Node.js

```bash
# 使用 nvm 管理版本
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
source ~/.zshrc
nvm install --lts

# 配置 npm 镜像
npm config set registry https://registry.npmmirror.com
```

### Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.zshrc
rustup update
```

## Docker

```bash
# 安装 Docker
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER

# 配置镜像加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 常用工具

### 终端复用器 tmux

```bash
# 安装
sudo apt install -y tmux

# 配置文件 ~/.tmux.conf
cat > ~/.tmux.conf << 'EOF'
# 启用鼠标支持
set -g mouse on

# 增大滚动缓冲区
set -g history-limit 50000

# 修改前缀键为 Ctrl+a（可选）
# set -g prefix C-a

# 更友好的分割快捷键
bind | split-window -h
bind - split-window -v

# 重载配置
bind r source-file ~/.tmux.conf \; display-message "Config reloaded"
EOF
```

### 文件搜索与查看

```bash
# ripgrep (rg) - 比 grep 更快
sudo apt install -y ripgrep

# fd - 比 find 更友好的文件搜索
sudo apt install -y fd-find

# bat - 带语法高亮的 cat
sudo apt install -y bat

# lsd - 更美观的 ls
sudo apt install -y lsd
```

### 系统监控

```bash
# htop - 交互式进程查看器
sudo apt install -y htop

# btop - 更现代化的监控工具
sudo apt install -y btop

# nvtop - GPU 监控（NVIDIA GPU）
sudo apt install -y nvtop

# iotop - 磁盘 IO 监控
sudo apt install -y iotop

# iftop - 网络流量监控
sudo apt install -y iftop
```

## 代理配置

```bash
# 在 ~/.zshrc 中添加
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7890

# 设置 no_proxy 绕过内网地址
export no_proxy=localhost,127.0.0.1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,.local
```

## 一键初始化脚本

将以上配置整合为一个脚本，方便新机器一键执行：

```bash
#!/bin/bash
set -e

echo "=== 开发机初始化脚本 ==="

# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装基础工具
sudo apt install -y build-essential cmake gdb git curl wget unzip tar \
  htop tmux ripgrep fd-find tree jq net-tools openssh-server zsh

# 安装 Oh My Zsh
if [ ! -d "$HOME/.oh-my-zsh" ]; then
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
fi

# 配置 Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase true

# 生成 SSH Key（如果不存在）
if [ ! -f "$HOME/.ssh/id_ed25519" ]; then
  ssh-keygen -t ed25519 -C "your.email@example.com" -N ""
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519
fi

echo "=== 初始化完成 ==="
echo "请手动执行: chsh -s $(which zsh)"
```

## 总结

以上配置覆盖了开发机的基础环境搭建。实际使用时根据具体需求（AI 开发、后端开发、前端开发等）增减相应工具链。建议将个性化配置（如 dotfiles）托管到 GitHub，方便多台机器同步。

<!--more-->
