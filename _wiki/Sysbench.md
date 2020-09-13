---
layout: article
title: Sysbench
tags: []
key: 2bfc99e4-c49d-4a37-a15b-4c15d9cb540e
---

Sysbench 是一个数据库压测工具。

<!--more-->


### 安装

在系统 CentOS 上 root 账号使用如下命令安装: 

```bash
# 下载包
wget https://github.com/akopytov/sysbench/archive/1.0.zip -O "sysbench-1.0.zip"
unzip sysbench-1.0.zip
cd sysbench-1.0
# 安装依赖
yum install automake libtool –y
# 安装
./autogen.sh
./configure
export LD_LIBRARY_PATH=/usr/local/mysql/include #这里换成机器中mysql路径下的include
make
make install
# 检查是否安装成功
sysbench --version
```

测试脚本所在的目录默认为: /usr/share/sysbench/tests/include/oltp_legacy/

### 语法

语法格式为:

```bash
sysbench [options]... [testname] [command]
```

#### options

测试相关的信息。


| 可选值 | 含义 |
| --- | --- |
| --mysql-host | mysql 地址 |
| --mysql-port | mysql 端口 |
| --mysql-user | mysql 账号 |
| --mysql-password | mysql 密码 |
| --oltp-test-mode | 执行模式: simple(查)、nontrx(增改查)、complex(增删改查，默认值) |
| --oltp-tables-count | 测试表数量 |
| --oltp-table-size | 测试表数据量 |
| --threads | 并发连接数 |
| --time | 测试执行时间 |
| --report-interval | 生成报告的时间 |

#### testname

指定需要执行的脚本。

#### command


| 可选值 | 含义 |
| --- | --- |
| prepare | 准备数据 |
| run | 执行测试 |
| cleanup | 清理数据 |

### 示例

准备数据:

```bash
sysbench ./tests/include/oltp_legacy/oltp.lua --mysql-host=192.168.10.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --oltp-test-mode=complex --oltp-tables-count=10 --oltp-table-size=100000 --threads=10 --time=120 --report-interval=10 prepare
```

执行测试:

```bash
sysbench ./tests/include/oltp_legacy/oltp.lua --mysql-host=192.168.10.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --oltp-test-mode=complex --oltp-tables-count=10 --oltp-table-size=100000 --threads=10 --time=120 --report-interval=10 run >> /home/test/mysysbench.log
```

清理数据:

```bash
sysbench ./tests/include/oltp_legacy/oltp.lua --mysql-host=192.168.10.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 cleanup
```
