---
layout: article
title: Python 包缺失
tags: []
key: 92b31f5a-7fd1-47ed-9563-f6450ab73878
---

Python 安装包或者引入包到项目中出现的问题.

<!--more-->

## `pillow` 包问题

报错：`ModuleNotFoundError: No module named 'PIL'`
解决方法：安装 `pillow` 包


## `lxml` 包问题(windows)

解决方法

到[网址](https://www.lfd.uci.edu/~gohlke/pythonlibs)下载对应的 `whl`：

```
# Python 3.8 版本，Python 32 位
lxml‑4.5.0‑cp38‑cp38‑win32.whl
```

在命令行安装:

```bash
pip install lxml‑4.5.0‑cp38‑cp38‑win32.whl
```

## `pycurl` 包问题

报错：` Curl is configured to use SSL, but we have not been able to determine which SSL backend it is using. Please see PycURL documentation for how to specify the SSL backend manually.`

解决办法：

```bash
export PYCURL_SSL_LIBRARY=openssl
# mac 下
export PYCURL_SSL_LIBRARY=openssl
export LDFLAGS=-L/usr/local/opt/openssl/lib
export CPPFLAGS=-I/usr/local/opt/openssl/include
```
