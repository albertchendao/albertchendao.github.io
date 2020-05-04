---
layout: article
title: poetry
tags: [Wiki]
key: 
---

Poetry 是 Python 的一个虚拟环境和依赖管理工具，并且提供了打包和发布功能。

## 安装

通过命令行(必须先安装 python):

```bash
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python
```

MAC OS 下:

```bash
brew install poetry
```

参考[官网](https://python-poetry.org/docs/)

## 使用

```bash
# 已有项目初始化 会生成 pyproject.toml 文件
poetry init
# 创建新项目
poetry new project_name
# 安装新环境 必须有 pyproject.toml 文件
poetry install
# 进入环境
poetry shell
# 在环境中执行命令
poetry run python app.py
# 安装包  开发环境使用 --dev 参数
poetry add flask
# 展示包
poetry show --tree
# 更新版本
poetry update [dep_name]
# 让 poetry 使用 python3
poetry env use python3.7
# 设置虚拟环境为项目里 .venv 目录
poetry config --list
poetry config virtualenvs.in-project true
# 从已有的 requirements.txt 导入
cat requirments.txt | perl -pe 's/([<=>]+)/:$1/g' | xargs -n 1 -I {} echo "poetry add '{}'"
```

添加国内的 pypi 源:

```bash
# pyproject.toml 添加
[[tool.poetry.source]]
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple/"
```
