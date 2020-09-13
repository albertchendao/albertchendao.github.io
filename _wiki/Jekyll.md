---
layout: article
title: Jekyll
tags: []
key: 5e980064-0247-49aa-848e-dc7a39bfc475
---

Jekyll 的使用.

<!--more-->

## MAC 安装 Jekyll

首先安装 rbenv, 这个可以本地切换多个版本的 ruby:

```bash
brew install rbenv
```

然后安装最新版本的 ruby:

```bash
rbenv install --list
rbenv install 2.6.3
```

然后安装 jekyll:

```bash
gem install jekyll
```

就可以执行 jekyll 相关的命令了:

```bash
jekyll build
# => 当前文件夹中的内容将会生成到 ./_site 文件夹中。

$ jekyll build --destination <destination>
# => 当前文件夹中的内容将会生成到目标文件夹<destination>中。

$ jekyll build --source <source> --destination <destination>
# => 指定源文件夹<source>中的内容将会生成到目标文件夹<destination>中。

$ jekyll build --watch
# => 当前文件夹中的内容将会生成到 ./_site 文件夹中，
#    查看改变，并且自动再生成。

$ bundle exec jekyll serve
```

## 搭建博客

从 [模板项目](https://github.com/kitian616/jekyll-TeXt-theme/tree/master/test) 复制出一个项目。

在终端中进入项目根目录。

安装依赖包:

```bash
bundle install --path vendor/bundle
```

启动项目:

```bash
bundle exec jekyll serve
```

本地访问项目: `http://localhost:4000`

## GitHub 集成

* 在 GitHub 建一个公开项目，项目名使用 `{username}.github.io`，在设置里配置好启用 GitHub Page。
* 在项目根目录下执行: `git init` 初始化成 git 项目，并 `git add .`, `git commit`
* 推送到 `{username}.github.io` 仓库即可，GitHub 会自动 build 出需要的静态页面
* 构建成功后就能直接访问 `http://{username}.github.io`

## 问题整理

## build 失败

有时候 `git push` 之后好长时间页面没有变化，可能是构建失败，可以去 GitHub 注册邮件找下是否有构建失败的邮件，里边会详细说明为什么失败

### 主题找不到

`_config.yml` 配置中可以修改 `theme: {theme-name}` 来切换主题，但是注意 GitHub 默认只支持推荐的主题(即仓库 setting 中 choose theme 推荐的)，如果要使用自定义主题需要其他方法:

1. 去掉 `theme: {theme-name}` 配置，将主题下载下来，集成到自己项目目录里
2. 如果主题是放在 GitHub 上的，可以使用 `remote_theme: {username}/{theme-name}` 替换掉 `theme: {theme-name}`, 但是这样本地 build 会失败，需要注意下

### 自定义域名

GitHub Page 支持自定义域名，首先要买个域名，然后在 GitHub 和域名提供商两边做下配置。

#### GitHub

仓库的 setting 中，设置下域名，然后保存，会在项目根目录下生成一个 CNAME 文件，里边就是设置的域名

#### 域名提供商

添加下资源记录:

```bash
# 4个 A 记录
@ A 1h 185.199.108.153
@ A 1h 185.199.109.153
@ A 1h 185.199.110.153
@ A 1h 185.199.111.153
# 2个 CNAME 记录
www CNAME 1h {username}.github.io
@ CNAME 1h {username}.github.io
```
