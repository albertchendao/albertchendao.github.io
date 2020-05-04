---
layout: article
title: pipenv
tags: [Wiki]
key: 
---

`python` 有使用 `pip` 进行包管理，但是包是所有项目共享的，后来又出现了 `virtualenv` 来分隔各个项目的依赖包。而新出现的 `pipenv` 就是用来替代这些的工具，每个项目单独一个包空间，管理自己的依赖，并且添加了更多的特性，详细的可以查看官网。

### 安装

```bash
pip install pipenv
```

MAC 下可以使用 `brew`:

```bash
brew install pipenv
```

### 使用

```bash
# 查看当前项目是否有创建环境
pipenv --venv
# 创建一个 python3 的环境
pipenv --three
# 创建一个 python3.6 的环境
pipenv --python 3.6
# 激活当前环境
pipenv shell
# 安装 django 到当前环境
pipenv install django
# 安装指定版本
pipenv install django==1.11
# 卸载包
pipenv uninstall django
# 加载系统 python 包
pipenv  --site-packages
# 查看当前环境包依赖
pipenv graph
# 退出当前环境
exit
```

### 切换国内源

默认使用官网源，下载速度比较慢，可以改成国内的源。

```bash
阿里云：http://mirrors.aliyun.com/pypi/simple/
豆瓣：http://pypi.douban.com/simple/
清华大学：https://pypi.tuna.tsinghua.edu.cn/simple/
中国科学技术大学：https://pypi.mirrors.ustc.edu.cn/simple/
```

切换目录到项目文件根目录
查看 Pipfile 的内容： cat Pipfile

```bash
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
flask = "*"
requests = "*"
wtforms = "*"
flask-sqlalchemy = "*"
cymysql = "*"
flask-login = "*"

[dev-packages]

[requires]
python_version = "3.7"
```

修改 source 下的 url
