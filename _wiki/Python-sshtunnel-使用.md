---
layout: article
title: Python sshtunnel 使用
tags: []
key: 2e69fbc8-cb77-4f80-85bf-5456673ed0da
---

`sshtunnel` 通过名称就可以明白它的作用是建立 SSH 隧道.在远程服务器只开放了 SSH 链接的时候,可以通过 SSH 隧道来访问服务器的其他端口.

<!--more-->

# 依赖

* Python 3.6
* sshtunnel 0.1.4

# 非跳转连接

当需要访问远程服务器的特定端口,而远程服务器又只开放了 SSH 端口(默认 22) 时,就可以使用 `sshtunnel` 来将本地的端口通过 SSH 端口转发到远程的特定端口.

```bash
----------------------------------------------------------------------

                            |
-------------+              |    +----------+
    LOCAL    |              |    |  REMOTE  | :22 SSH
    CLIENT   | <== SSH ========> |  SERVER  | :8080 web service
-------------+              |    +----------+
                            |
                         FIREWALL (only port 22 is open)

----------------------------------------------------------------------
```

示例代码:

```python
from sshtunnel import SSHTunnelForwarder

server = SSHTunnelForwarder(
    'pahaz.urfuclub.ru',   # 远程服务器地址
    ssh_username="pahaz",  # 远程服务器账号
    ssh_password="secret", # 远程服务器密码
    remote_bind_address=('127.0.0.1', 8080)  # 本地服务器 ip 和 端口
)

server.start()

print(server.local_bind_port)  # 输出本地端口
# 访问本地的端口会转发到远程服务器上

server.stop()
```

# 跳转连接

通过 SSH 连接跳板机然后访问远程的私有服务器.

```bash
----------------------------------------------------------------------

                            |
-------------+              |    +----------+               +---------
    LOCAL    |              |    |  REMOTE  |               | PRIVATE
    CLIENT   | <== SSH ========> |  SERVER  | <== local ==> | SERVER
-------------+              |    +----------+               +---------
                            |
                         FIREWALL (only port 443 is open)

----------------------------------------------------------------------
```

示例代码:

```python
import paramiko
from sshtunnel import SSHTunnelForwarder

with SSHTunnelForwarder(
    (REMOTE_SERVER_IP, 443),
    ssh_username="",
    ssh_pkey="/var/ssh/rsa_key",
    ssh_private_key_password="secret",
    remote_bind_address=(PRIVATE_SERVER_IP, 22),
    local_bind_address=('0.0.0.0', 10022)
) as tunnel:
    client = paramiko.SSHClient()
    client.load_system_host_keys()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    client.connect('127.0.0.1', 10022)
    # do some operations with client session
    client.close()

print('FINISH!')
```





