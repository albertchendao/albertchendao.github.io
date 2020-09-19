---
layout: article
title: sdkman 使用
tags: []
key: 92b12714-4cd2-43c0-877a-3808d58737a3
---

SDKMAN 是管理多个软件开发工具包的并行版本的工具.它提供了一个方便的命令行界面（CLI）和API来安装,切换,删除和列出sdk相关信息.

<!--more-->

## 安装与卸载

```bash
# 安装
curl -s "https://get.sdkman.io" | bash
# 安装完后执行
source "$HOME/.sdkman/bin/sdkman-init.sh"
# 查看版本
sdk version
# 自定义安装的 SDK 目录
export SDKMAN_DIR="/usr/local/sdkman" && curl -s "https://get.sdkman.io" | bash
# 升级 sdkman
sdk selfupdate
# 强制升级 sdkman
sdk selfupdate force
# 卸载
tar zcvf ~/sdkman-backup_$(date +%F-%kh%M).tar.gz -C ~/ .sdkman
$ rm -rf ~/.sdkman
# 卸载后 从 zshrc 等中删除如下
#THIS MUST BE AT THE END OF THE FILE FOR SDKMAN TO WORK!!!
[[ -s "/home/dudette/.sdkman/bin/sdkman-init.sh" ]] && source "/home/dudette/.sdkman/bin/sdkman-init.sh"
```

## 使用

```bash
# 列出支持的软件
sdk list
sdk list gradle
sdk list java
# 安装
sdk install java
# 安装指定版本
sdk install java 8.0.252-zulu
# 通过本地包安装
sdk install groovy 3.0.0-SNAPSHOT /path/to/groovy-3.0.0-SNAPSHOT
sdk install java 8.0_181-oracle /Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home
# 卸载
sdk uninstall scala 2.11.6
# 选择版本
sdk use scala 2.12.1
# 设置默认版本
sdk default scala 2.11.6
# 查看当前使用的版本
sdk current java
```
