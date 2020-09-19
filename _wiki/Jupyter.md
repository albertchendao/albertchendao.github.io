---
layout: article
title: Jupyter
tags: []
key: d1f70110-1037-4b26-8b5f-9564f0290212
---

Jupyter Notebook 是一个交互式的应用程序,可以在网页页面中直接编写代码、运行、展示运行结果,非常适合编写编程相关的说明文档.

<!--more-->

## 安装

依赖版本: Python 3.3 以上 或 Python 2.7

### Anaconda 安装

直接使用如下命令:

```bash
conda install jupyter notebook
```

### Pip 安装

Python 3 版本下:

```bash
pip3 install --upgrade pip
pip3 install jupyter
```

Python 2 版本下:

```bash
pip install jupyter
```

## 配置

第一次时执行如下命令会生成一个默认的配置文件,后续再执行这个命令会提示是否使用默认配置覆盖之前的配置:

```bash
jupyter notebook --generate-config
```

修改配置文件:

```bash
vim ~/.jupyter/jupyter_notebook_config.py
```

相关的配置:

```bash
# 文档的保存地址
c.NotebookApp.notebook_dir = '/Users/albert/Dropbox/notebook'
```

## 运行命令

```bash
# 启动命令  默认使用 http://localhost:8888
jupyter notebook
# 指定端口启动
jupyter notebook --port <port_number>
# 不打开浏览器启动
jupyter notebook --no-browser
```


