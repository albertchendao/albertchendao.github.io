---
layout: article
title: 修改 Host 不生效
tags: []
key: bf7bd767-9dc8-4149-8414-74bd99e60596
---

开发调试的时候经常需要修改 hosts 文件,将特定域名指向特定的 ip,但是 MAC 上经常不会生效.

### 原因

因为服务器设置了 [keep-alive](https://zh.wikipedia.org/wiki/HTTP%E6%8C%81%E4%B9%85%E8%BF%9E%E6%8E%A5),次要原因是存在浏览器 DNS 缓存和系统 DNS 缓存.

服务器在响应头设置了 Connection: keep-alive （一般的网页都会设置 keep-alive,保持长连接,避免多次连接产生网络消耗）之后,客户端会跟服务器保持长连接,只要长连接不断开,页面在请求的时候就不会重新解析域名！

我们可以这样来测试:

1. 打开一个你至少两分钟没有打开的浏览器（你也可以关闭掉你的浏览器,然后重新打开,记得把所有的 tab 都关了,除了当前 tab）
2. 在 hosts 添加 `·127.0.0.1 www.taobao.com`
3. 新开 tab,打开 www.taobao.com,是不是进不去了 <这里说明 hosts 修改生效了>
4. 注释掉刚才hosts修改,`# 127.0.0.1 www.taobao.com` ,再打开 www.taobao.com,很好,正常打开了 <这里说明 hosts 修改也生效了>
5. 去掉注释符,127.0.0.1 www.taobao.com ,再打开 www.taobao.com,依然可以访问！！！
6. Chrome 中进入 chrome://net-internals/#sockets,可以看到淘宝首页中很多域名都是与服务器保持着长连接,点击上方的 close idle sockets 按钮,可以关闭所有的长连接
7. 此时,再去访问 www.taobao.com,是不是进不去了！

还有其他原因可能会造成失效:

1. 如果浏览器使用了代理工具,修改 Hosts 也不会生效.这里是因为,浏览器会优先考虑代理工具（如添加 pac 文件、SwitchySharp等）的代理,建议调试的时候先关闭这些代理.
2. 使用 pac 文件代理有的时候部分文件的代理不生效,应该是 pac 对应的代理服务器上,做了部分处理.
3. 部分浏览器也有 DNS 缓存,如 chrome(chrome://dns),这是为什么重启浏览器也不生效的原因,一般设定时间为 60s (如 Firefox).
4. 浏览器有DNS缓存,系统也会存在 DNS 缓存,有的时候即便在 chrome://dns 清空了浏览器 DNS 缓存,依然不生效,是因为系统 DNS 缓存还未刷新.windows 下: `ipconfig /flushdns`,linux 下: `/etc/rc.d/init.d/nscd restart` 或者新版本 `/etc/init.d/nscd restart`, MAC 下: `lookupd -flushcache` 或者新的 `type  dscacheutil -flushcache` 或者更新的: `sudo killall -HUP mDNSResponder
`

### 强制生效

可以采用如下方法强制使修改的 host 生效:

1. 重启浏览器,所有的连接（包括长连接）都会断开,自然就生效了
2. 隐私模式打开,隐私模式下不会复用 TCP 连接,新开连接的时候,会重新解析 DNS 域名,自然也生效了
3. iHosts 管理器在 Mac 下生效,在修改 hosts 文件的时候,会重启网络服务
4. 等一会儿,要稍微等久一点,keep-alive 的默认设置是 120s


