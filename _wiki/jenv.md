---
layout: article
title: jenv
tags: []
key: a1cbd33e-4d9a-436c-bd55-97bd49f5b231
---

## 安装

[官网](https://www.jenv.be/)

安装步骤:

```shell
# MAC 下安装
brew install jenv
# shell 配置,这里是 zsh
echo 'export PATH="$HOME/.jenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(jenv init -)"' >> ~/.zshrc
# 从 Oracle 下载 JDK 进行安装,然后加入到 jenv
jenv add /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/
```

## 问题

### maven 未使用 jenv 的配置

安装了 `JDK13` 后,`maven` 的 `Java Version` 就变成 `JDK13` 了:

![200f80845e1c9d28a1a59358d260d750.png](evernotecid://E33F44EF-E267-42C5-8ACC-CF97D470728E/appyinxiangcom/8490036/ENResource/p1745)

解决办法: 执行命令 `jenv enable-plugin maven`

[参考链接](https://stackoverflow.com/questions/30602633/maven-ignoring-jenv-settings)
