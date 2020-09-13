---
title: Redis Cluster 安装
tags: []
key: 417062ec-3016-46a1-898e-4419ab85fdc2
---

<!--more-->

## 安装包

下载 `redis-4.0.14.tar.gz` 包。

## 系统配置

1. 调整内核参数:

    ```bash
    # /etc/sysctl.conf 
    vm.overcommit_memory=1
    ```

2. 关闭 THP

    ```bash
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
    ```

3. 调整单用户最大打开文件数

    ```bash
    # /etc/security/limits.conf
    *              soft    nofile          65535
    *              hard    nofile          65535
    ```

## 安装

1. 解压 `redis-4.0.14.tar.gz` 到 `/opt/redis` 目录下
2. 修改配置文件，`bin/conf/template/` 目录下有对应的配置文件模板，只需修改 `port` 和 `ansible_host` 即可

    ```bash
    redis_cluster.conf.template  # 集群配置文件
    redis_sentinel.conf.template # 哨兵配置文件
    redis_standalone.conf.template  # 单实例配置文件
    ```

3. 配置监控，用于 redis 进程挂掉时进行自动拉起:
    * 配置 crontab 定时任务

        ```bash
        */1 * * * * /bin/bash /opt/redis/monitor/bin/redis_monitor.sh execute
        ```

    * 配置监控文件，`monitor/conf/template.txt` 文件是配置模板，真实配置文件为 `monitor/conf/redis_discovery.txt`

### 安装集群

1. 解压 `redis-4.0.14.tar.gz` 到 `/opt/redis` 目录下
2. 在 `/opt/redis` 下创建一个 `data` 目录用于存放数据文件，一个 `conf` 目录用于存放配置
3. 复制配置文件 `bin/conf/template/redis_cluster.conf.template` 到 `conf` 目录，修改名称为 `redis_6379.conf`
4. 修改 `redis_6379.conf` 文件中的双大括号括起来的配置项，端口使用 6379，`data` 绝对路径配置到 `dir` 配置项上，注意 `bind` 使用内网 ip，使用 `0.0.0.0` 访问可能有问题
5. 复制 `redis_6379.conf` 文件为 `redis_6380.conf`  `redis_6381.conf`，并修改对应的端口
6. 启动三个实例：

    ```bash
    ./src/redis-server ./conf/redis_6379.conf
    ./src/redis-server ./conf/redis_6380.conf
    ./src/redis-server ./conf/redis_6381.conf
    ```

7. 需要用到 `ruby`，安装一下依赖包：

    ```bash
    yum install ruby
    yum install rubygems
    gem install redis 
    ```

8. 创建集群：

    ```bash
    ./src/redis-trib.rb create --replicas 0 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381
    ```

9. 测试，注意如果不加 `-c` 参数会出现 `(error) MOVED 5798 127.0.0.1:6380`

    ```bash
    ./src/redis-cli -c -h 127.0.0.1 -p 6379
    cluster info
    set name mafly
    get name
    del name
    ```

## 相关操作

### 主从

```bash
# 主从同步 从节点上执行
slaveof master_ip master_port
# 取消主从同步 从节点上执行
slaveof no one
```

### 集群

新建集群步骤:

1. nodes 启用 cluster 配置
2. nodes 加入集群
3. 分配 slot
4. 为 master 节点添加 slave 节点

通过脚本管理集群(`redis-cli5` 路径: `redis/bin/redis-cli5`):

1. 准备启用了 cluster 配置的 nodes 节点(安装redis步骤中安装配置的cluster节点的redis)
2. 创建集群

    ```bash
    redis-cli5 --cluster create --cluster-replicas 1 ip:port ip:port ...
    redis-cli5 --cluster help
    ```

3. 集群扩容

    ```bash
    # 新节点加入集群
    redis-cli5 --cluster add-node new_host:new_port existing_host:existing_port
    # slot 重分片
    redis-cli5 --cluster reshard ip:port(集群任意节点ip端口) 
    # 为扩容的新节点添加 slave
    ## 在 slave 上执行
    cluster replicate masternodeid
    ```

4. 集群缩容

    ```bash
    # 将要下线节点上的slot迁移到其他节点，保证下线节点上没有slot
    redis-cli5 --cluster reshard ip:port(集群任意节点ip端口) 
    # 集群忘记该节点
    redis-cli5 --cluster del-node ip:port(集群任意节点ip端口)  node_id
    ```

5. slot 自动平衡

    ```bash
    redis-cli5 --cluster rebalance ip:port(集群任意节点ip端口)
    ```

6. 集群手动切换主从

    ```bash
    # slave 节点上
    cluster failover
    ```
