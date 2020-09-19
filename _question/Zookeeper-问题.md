---
layout: article
title: Zookeeper 问题
tags: []
key: 7cdf76a5-3ebf-4e70-9173-16a5cb81d813
---

Zookeeper 使用中的问题.

<!--more-->

tags: []
---

## 问题

在 Mac 上 zookeeper 后连接不上:

```bash
# 安装
brew install zookeeper
# 启动
zkServer start
# 连接
zkCli
# 查询异常
[zk: localhost:2181(CONNECTING) 0] ls /
KeeperErrorCode = ConnectionLoss for /
```

默认的安装目录是: `/usr/local/Cellar/zookeeper/3.5.8`
默认的启动目录是: `/usr/local/etc/zookeeper/`
从配置文件 `log4j.properties` 发现日志目录是: `/usr/local/var/log/zookeeper/`
有异常日志:

```bash
2020-07-21 18:55:11 NIOServerCnxnFactory [ERROR] Thread Thread[main,5,main] died
java.lang.NoSuchMethodError: java.nio.ByteBuffer.clear()Ljava/nio/ByteBuffer;
	at org.apache.jute.BinaryOutputArchive.stringToByteBuffer(BinaryOutputArchive.java:77)
	at org.apache.jute.BinaryOutputArchive.writeString(BinaryOutputArchive.java:107)
	at org.apache.zookeeper.data.Id.serialize(Id.java:51)
	at org.apache.jute.BinaryOutputArchive.writeRecord(BinaryOutputArchive.java:123)
	at org.apache.zookeeper.data.ACL.serialize(ACL.java:52)
	at org.apache.zookeeper.server.ReferenceCountedACLCache.serialize(ReferenceCountedACLCache.java:136)
```

## 原因

安装的 zookeeper 是 3.5.8 版本, 使用的 jdk9 以上进行的编译, 而 jdk8 和 jdk9 的 NIO 有不兼容问题.

## 解决

安装 zookeeper 3.4 版本:

```bash
brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/6d8197bbb5f77e62d51041a3ae552ce2f8ff1344/Formula/zookeeper.rb
```

kafka 也会有问题, 安装可以运行的版本:

```bash
brew install --ignore-dependencies https://raw.githubusercontent.com/Homebrew/homebrew-core/6d8197bbb5f77e62d51041a3ae552ce2f8ff1344/Formula/kafka.rb
```


## 参考链接

1. [Kafka with Zookeeper 3.5.7 Crash NoSuchMethodError: java.nio.ByteBuffer.flip()](https://stackoverflow.com/questions/60612999/kafka-with-zookeeper-3-5-7-crash-nosuchmethoderror-java-nio-bytebuffer-flip)
