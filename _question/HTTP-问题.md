---
layout: article
title: HTPP 问题
tags: []
key: d0add1f1-60d7-4f99-91f4-70d6e9579ce3
---

七牛分片下载失败问题.

<!--more-->

## 分片下载

### 问题

根据七牛的[文档](https://developer.qiniu.com/kodo/manual/1232/download-process)可以在请求头中添加 `Range: bytes=<first-byte-pos>-<last-byte-pos>` 来支持分片下载.

但是实际使用 `httpie`, `requests` 请求的时候发现不起作用, 下载的字节数总是超过 `Range` 指定的范围.

### 排查

问了七牛, 他们给了一个正常的 `Range` 生效的 HTTP 请求示例:

```bash
# curl -voa -H 'Range:bytes=0-20' https://pres-ssp.iapp.jiguang.cn/txt/20200602010529/0_TOP_1000_20160101-20200530.txt?attname=0_TOP_1000_20160101-20200530.txt&e=1591066098&token=pNf8Uh3lsdBw4BGIUE1tgWT8dPKDDyzuOGEgtEJi:BEvt3deU7TOY0IUa_pPUKH9GXUE=

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying 113.96.181.177...
* TCP_NODELAY set
* Connected to pres-ssp.iapp.jiguang.cn (113.96.181.177) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/cert.pem
  CApath: none
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
} [230 bytes data]
* TLSv1.2 (IN), TLS handshake, Server hello (2):
{ [108 bytes data]
* TLSv1.2 (IN), TLS handshake, Certificate (11):
{ [2949 bytes data]
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
{ [300 bytes data]
* TLSv1.2 (IN), TLS handshake, Server finished (14):
{ [4 bytes data]
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
} [37 bytes data]
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
} [1 bytes data]
* TLSv1.2 (OUT), TLS handshake, Finished (20):
} [16 bytes data]
* TLSv1.2 (IN), TLS change cipher, Client hello (1):
{ [1 bytes data]
* TLSv1.2 (IN), TLS handshake, Finished (20):
{ [16 bytes data]
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: C=CN; ST=Guangdong province; L=Shenzhen; O=Shenzhen HeXunHuaGu Information Technologies Co.Ltd; CN=*.iapp.jiguang.cn
*  start date: May 20 00:00:00 2020 GMT
*  expire date: May 20 12:00:00 2022 GMT
*  subjectAltName: host "pres-ssp.iapp.jiguang.cn" matched cert's "*.iapp.jiguang.cn"
*  issuer: C=US; O=DigiCert Inc; CN=DigiCert SHA2 Secure Server CA
*  SSL certificate verify ok.
> GET /txt/20200602010529/0_TOP_1000_20160101-20200530.txt?attname=0_TOP_1000_20160101-20200530.txt&e=1591066098&token=pNf8Uh3lsdBw4BGIUE1tgWT8dPKDDyzuOGEgtEJi:BEvt3deU7TOY0IUa_pPUKH9GXUE= HTTP/1.1
> Host: pres-ssp.iapp.jiguang.cn
> User-Agent: curl/7.54.0
> Accept: */*
> Range:bytes=0-20
>
< HTTP/1.1 206 Partial Content
< Server: Tengine
< Content-Type: text/plain
< Content-Length: 21
< Connection: keep-alive
< Via: cache17.l2cn1851[36,206-0,M], cache36.l2cn1851[37,0], cache16.cn1368[53,206-0,M], cache16.cn1368[55,0]
< Date: Tue, 02 Jun 2020 01:51:08 GMT
< Accept-Ranges: bytes
< Access-Control-Allow-Origin: *
< Access-Control-Expose-Headers: X-Log, X-Reqid
< Access-Control-Max-Age: 2592000
< Cache-Control: public, max-age=31536000
< Content-Disposition: attachment; filename="0_TOP_1000_20160101-20200530.txt"; filename*=utf-8''0_TOP_1000_20160101-20200530.txt
< Content-Range: bytes 0-20/1122135153
< Content-Transfer-Encoding: binary
< Etag: "lhR0NWeHMLyc6HdOKLEEe0wbyzs9"
< Last-Modified: Mon, 01 Jun 2020 17:10:18 GMT
< Vary: Accept-Encoding
< X-Log: X-Log
< X-M-Log: QNM:xs1183;SRCPROXY:xs489;SRC:7;SRCPROXY:7;QNM3:7
< X-M-Reqid: 8IYAAFgZnD0RlxQW
< X-Private: 1
< X-Qiniu-Zone: 0
< X-Qnm-Cache: RawProxy
< X-Reqid: wvsAAABl2D0RlxQW
< X-Svr: IO
< X-Cache: MISS TCP_MISS dirn:-2:-2
< X-Cache: MISS TCP_MISS dirn:-2:-2
< Timing-Allow-Origin: *
< EagleId: 7160b5a415910626686472895e
<
{ [21 bytes data]
100    21  100    21    0     0    106      0 --:--:-- --:--:-- --:--:--   107
* Connection #0 to host pres-ssp.iapp.jiguang.cn left intact
```

也就是说正常的分片下载返回的 HTTP 状态码应该是 `206`, 但是 `httpie`, `requests` 返回的状态码始终是 `200`.

调试发现是因为添加了一些默认请求头引起的, 经过测试如下:

```bash
# 正常分片下载
curl -voa -H 'Range:bytes=0-20' {url}
# 正常分片下载
curl -voa -H 'Accept-Encoding:identity' -H 'Range:bytes=0-20' {url}
# 失败分片下载
curl -voa -H 'Accept-Encoding:gzip, deflate' -H 'Range:bytes=0-20' {url}
# 失败分片下载
curl -voa -H 'Connection:keep-alive' -H 'Range:bytes=0-20' {url}
```

### 解决方案

所以如果要使 `Range` 正常生效, 需要重置 `Accept-Encoding`, `Connection` 请求头.

在 `httpie` 中使用如下请求:

```bash
http -v GET {url} \
Accept-Encoding: \
Connection: \
'Range: bytes=0-20'
```

在 `requests` 中使用如下方法:

```bash
headers = {'Accept-Encoding': '', 'Connection': '', 'Range': 'bytes=%d-%d' % (0, 20)}
with requests.get(url, stream=True, verify=False, headers=headers) as r:
    print(r.status_code)
    with open(file_path, "ab") as f:
        for chunk in r.iter_content(chunk_size=1024):
            if chunk:
                temp_size += len(chunk)
                f.write(chunk)
                f.flush()

                ##这是下载实现进度显示####
                done = int(50 * temp_size / total_size)
                sys.stdout.write("\r[%s%s] %d%%" % ('█' * done, ' ' * (50 - done), 100 * temp_size / total_size))
                sys.stdout.flush()
print()  # 避免上面\r 回车符
```

### 参考链接

* [HTTP Range Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Range)
* [httpie 默认请求头](https://github.com/jakubroztocil/httpie#http-headers)
* [StackoverFlow 如何重置 httpie 请求头](https://stackoverflow.com/questions/28978632/remove-default-http-headers-from-httpies-request)
