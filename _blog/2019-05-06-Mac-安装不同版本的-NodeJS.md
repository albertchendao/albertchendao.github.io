---
layout: article
title: 安装不同版本的 NodeJS
tags: [FrontEnd]
key: 3296b067-2708-4095-8aad-4c58679df3d5
---

Mac 安装不同版本的 NodeJS.

<!--more-->

## 使用 `n`

### 安装

默认已经有 homebrew, 安装最新版本的 nodejs:

```bash
brew install nodejs
```

### 版本切换

安装工具 n:

```bash
node -v
sudo npm cache clean -f
sudo npm install -g n
sudo n stable
node -v
```

使用 n:

```bash
sudo n --lastest //最新版
sudo n --stable //稳定版
sudo n 4.x //4系列版本
sudo n 6.x //6系列版本
sudo n  //查看所有版本并选择
```

## 使用 `nvm`

### 安装

[官网](https://github.com/nvm-sh/nvm#installing-and-updating)

使用如下命令:

```bash
brew update
brew install nvm
mkdir ~/.nvm
```

因为使用的 `zsh`,还需要修改 `~/.zshrc`:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "/usr/local/opt/nvm/nvm.sh" ] && . "/usr/local/opt/nvm/nvm.sh"  # This loads nvm
[ -s "/usr/local/opt/nvm/etc/bash_completion.d/nvm" ] && . "/usr/local/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion
```

相关命令:

```bash
# 列出所有可安装的版本
nvm ls-remote
# 安装指定的版本,如 nvm install v8.14.0
nvm install <version>
# 卸载指定的版本
nvm uninstall <version>
# 列出所有已经安装的版本
nvm ls
# 切换使用指定的版本
nvm use <version>
# 显示当前使用的版本
nvm current
# 设置默认 node 版本
nvm alias default <version>
# 解除当前版本绑定
nvm deactivate
```
