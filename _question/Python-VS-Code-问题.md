---
layout: article
title: Python VS Code 问题
tags: []
key: 844b96b9-a596-40f9-b4cf-2d388380cfc0
---

使用 VS Code 编写 Python 代码的问题.

<!--more-->

## 不能使用项目里的包

在项目的 `setting.json` 配置中加上：

```json
"python.autoComplete.extraPaths": ["./path-to-your-code"]
```


## 不能使用 poetry 添加的包

使用 poetry 创建项目，用 vs code 打开后，import 包会出错。

在项目的 `setting.json` 配置中加上：

```json
"python.pythonPath": "/Users/albert/Library/Caches/pypoetry/virtualenvs/scrapydemo-DjAusZBP-py3.7/bin/python"
```

或者加入如下配置后使用 `> Python Select Interpreter` 选择指定的环境：

```json
"python.venvPath": "/Users/albert/Library/Caches/pypoetry/virtualenvs",
```