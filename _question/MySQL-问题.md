---
layout: article
title: MySQL 问题
tags: []
key: 359554c4-639a-496b-b866-61aa382c96f5
---

MySQL 使用中相关的问题.

<!--more-->

## Mac MySQL 启动报错

MacOS 通过 `brew` 安装 `mysql` 后启动报错:`ERROR! The server quit without updating PID file (/usr/local/var/mysql/bogon.pid).`

查看 `cat /usr/local/var/mysql/*.err|tail -100f` 发现如下 BUG：

```bash
2020-04-20T13:38:20.6NZ mysqld_safe Logging to '/usr/local/var/mysql/bogon.err'.
2020-04-20T13:38:20.6NZ mysqld_safe Starting mysqld daemon with databases from /usr/local/var/mysql
dyld: Library not loaded: /usr/local/opt/openssl/lib/libssl.1.0.0.dylib
  Referenced from: /usr/local/Cellar/mysql@5.7/5.7.24/bin/mysqld
  Reason: image not found
2020-04-20T13:38:20.6NZ mysqld_safe mysqld from pid file /usr/local/var/mysql/bogon.pid ended
```

解决方法是执行如下命令：

```bash
brew switch openssl 1.0.2s
```

[参考链接](https://stackoverflow.com/questions/59006602/dyld-library-not-loaded-usr-local-opt-openssl-lib-libssl-1-0-0-dylib)

## MySQL 编码

MySQL 使用的 `utf8` 编码，插入的时候报错: `Incorrect string value: '\xF0\x9F\x87\xA8\xF0\x9F...' for column 'model' at row 317`。

这个编码 `\xF0\x9F\x87\xA8\xF0\x9F` 就是 `utf8` 编码的 16 进制形式，转换成可读的字符串：

```java
    public static void main(String[] args) {
        String s = "\\xF0\\x9F\\x87\\xA8";
        String[] strs = s.split("\\\\x");

        byte[] values = new byte[strs.length - 1];
        for (int i = 1; i < strs.length; i++) {
            String str = strs[i];
            Integer v = Integer.valueOf(str, 16);
            System.out.println(Integer.toBinaryString(v));
            byte b = v.byteValue();
            values[i - 1] = b;
        }
        String f = new String(values);
        System.out.println(f);
    }
```

结果：

```bash
11110000
10011111
10000111
10101000
🇨
```

修改库使用 utf8mb4:

```sql
ALTER DATABASE iapp2 CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci ;
ALTER TABLE tb_mk_brand_store_visitor_number CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci ;

ALTER TABLE tb_monitor_favorite CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_monitor_mp_favorite CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_monitor_mp_repair_data CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_monitor_mp_repair_log CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_monitor_oversea_favorite CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_monitor_oversea_repair_data CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_monitor_oversea_repair_log CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_monitor_repair_data CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_monitor_repair_log CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_monitor_transform_log CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_app_change CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_app_change_log CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

ALTER TABLE tb_app CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_mp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tb_mp_info CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
