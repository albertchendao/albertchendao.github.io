---
layout: article
title: free 命令详解
tags: []
key: 820443a3-478f-42d8-b959-1ea8cf7b2fec
---

<!--more-->

## 用法

```bash
# 每隔 3s 刷新一次, 使用可读单位(KB, MB, GB)
free -h -s 3
```

## 输出

```bash
              total        used        free      shared  buff/cache   available
Mem:            15G        9.6G        240M        833M        5.8G        4.9G
Swap:            0B          0B          0B
```

|     字段      |                 含义                  |                  备注                  |
| :-----------: | :-----------------------------------: | :------------------------------------: |
|    Mem 行     |            内存的使用情况             |                                        |
|    Swap 行    |          交换空间的使用情况           |    通过硬盘扩展内存, 两者间换入换出    |
|   total 列    |  系统总的可用物理内存和交换空间大小   |                                        |
|    used 列    |    已经被使用的物理内存和交换空间     |                                        |
|    free 列    |  还有多少物理内存和交换空间可用使用   |                                        |
|   shared 列   |       被共享使用的物理内存大小        |                                        |
| buff/cache 列 | 被 buffer 和 cache 使用的物理内存大小 | 当 free 不够时可以被回收分配给应用程序 |
| available 列  |  还可以被应用程序使用的物理内存大小   |   available  = free + buffer + cache   |

buffer 在操作系统中指 buffer cache， 中文一般翻译为 "缓冲区"。
cache 在操作系统中指 page cache，中文一般翻译为 "页高速缓存"。

## 参考链接

1. [linux下free命令详解](https://www.cnblogs.com/ultranms/p/9254160.html)
