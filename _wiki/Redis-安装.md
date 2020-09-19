---
layout: article
title: Redis 安装
tags: []
key: 63c67aa2-a1a0-4d7e-ad55-f1df33c40df1
---

Redis 是一个高性能的 key-value 数据库.

<!--more-->

## 本地安装

### Mac

从 [官网链接](https://redis.io/download) 下载最新的稳定(stable)版本,找到下载的包,依次执行如下命令:

```bash
# 下载
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
# 解压
tar zxvf redis-4.0.9.tar.gz
# 移动
mv redis-4.0.9 /usr/local/
# 切换到目录
cd /usr/local/redis-4.0.9/
# 编译测试
sudo make test
# 编译,之后就可以使用 ./redis-4.0.9/src/redis-cli -h {} -p {} 命令
sudo make
# 编译安装
sudo make install
```

## 相关命令

### 启动

默认端口 6379, 启动方法:

```bash
redis-server
redis-server /etc/redis.conf
```

### 连接

```bash
./redis-cli -h localhost -p 6379
```

### 停止

```bash
# 客户端内执行
SHUTDOWN
# 关闭不了加个参数
SHUTDOWN NOSAVE
# 直接结束掉进程
kill redis
```
