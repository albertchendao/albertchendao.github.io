---
layout: article
title: HttpRunner
tags: []
Key: 685cab0a-8203-410c-8a2a-d6fad8bdacac
---

HttpRunner 是一款面向 HTTP(S) 协议的通用测试框架,只需编写维护一份 YAML/JSON 脚本,即可实现自动化测试、性能测试、线上监控、持续集成等多种测试需求.

<!--more-->

## 文档

[中文文档](https://cn.httprunner.org/)

## 安装

使用 `poetry`:

```bash
poetry add httprunner
```

安装的时候遇到:`socket.timeout: The read operation timed out`,所以改成如下命令进行安装:

```bash
poetry shell 
python --version
pip --version 
pip --default-timeout=1000 install httprunner 
poetry add httprunner
```

如果需要通过抓包生成测试用例,也需要安装 `har2case`:

```bash
poetry add har2case
```

## 用法

在浏览器调试模式下找到对应的接口,右键 `Copy -> Copy all as HAR`,将复制出来的内容保存为 `demo.har` 文件.

通过如下命令将 `demo.har` 转变为 `demo.yml` 测试用例配置文件:`har2case demo.har -2y`

执行测试用例:`hrun demo.yml`

运行的结果在当前目录下的 `reports` 目录

