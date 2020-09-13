---
layout: article
title: CORS 详解
tags: []
key: e5eb3808-d388-4aa6-b3eb-ff220a5119cc
---

`CORS`全称是"跨域资源共享"(Cross-origin resource sharing), 它允许浏览器向跨域(`origin`)服务器发出资源请求。

<!-- more -->

## 跨域请求

如果两个 URL 协议(`protocol`), 主机(`host`), 端口(`port`) 完全相同, 我们就称这两个 URL 同域(`origin`).

浏览器默认会阻止跨域(`origin`)的请求. 具体的说, 如果不做特殊处理, 在 `origin_one` 下的页面通过 `origin_two` 请求资源会被浏览器拦截(实际未发出请求), 浏览器会报如下的错误:

```bash
Access to XMLHttpRequest at 'origin_two/api' from origin 'oriing_one' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

跨域的如下行为会受到限制:

* `Cookie`、`LocalStorage`、`IndexDB`无法读取
* `DOM`无法获得
* `AJAX`请求不能发送

## CORS

`CORS`是 `HTTP`协议的一部分, 专门用来解决跨域的请求, 它实际是一组 `HTTP` 请求头, 规范了跨源下, 浏览器和服务器之间的请求流程, 包含如下请求头:

| 请求头                           | 含义                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| Access-Control-Allow-Origin      | 指示请求的资源能共享给哪些域                                 |
| Access-Control-Allow-Credentials | 当为 true 时, 允许客户端携带验证信息，例如 cookie 之类的     |
| Access-Control-Allow-Headers     | 用在对预请求的响应中，指示实际的请求中可以使用哪些 HTTP 头   |
| Access-Control-Allow-Methods     | 指定对预请求的响应中，哪些 HTTP 方法允许访问请求的资源       |
| Access-Control-Expose-Headers    | 指示哪些 HTTP 头的名称能在响应中列出                         |
| Access-Control-Max-Age           | 指示预请求的结果能被缓存多久                                 |
| Access-Control-Request-Headers   | 用于发起一个预请求，告知服务器正式请求会使用那些 HTTP 头     |
| Access-Control-Request-Method    | 用于发起一个预请求，告知服务器正式请求会使用哪一种 HTTP 请求方法 |
| Origin                           | 指示获取资源的请求是从什么域发起的                           |

浏览器将所有跨域请求分为两种, 处理方式不一样.

### 简单请求

简单请求必须满足如下情况:

- HTTP 请求方法必须是如下一种: `HEAD`, `GET`, `POST`
- 请求头必须不超过如下范围:
  - Accept
  - Accept-Language
  - Content-Language
  - Last-Event-ID
  -  Content-Type(只能为 application/x-www-form-urlencoded、multipart/form-data、text/plain)

对简单请求, 浏览器直接在请求头上加上 `Origin` 字段, 然后发送给服务器, 也就是说只有一个请求. 此时, 如果返回的响应头上没有 `Access-Control-Allow-Origin`字段, 浏览器就会报如上的 `CORS policy`问题; 如果服务器允许跨域就会返回 `Access-Control-Allow-Origin` 字段及正常的返回体.

### 非简单请求

简单请求以外的都是非简单请求, 浏览器会发送两次请求: 预检请求, 内容请求.

#### 预检请求

预检请求的 HTTP 请求方法为 `OPTIONS`, 并且附带上跨域请求资源时的相关信息, 例如:

```bash
OPTIONS /api HTTP/1.1
Origin: http://test.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization
Host: test.com
```

附带上了请求资源的域(`origin`), 请求方法(`Access-Control-Request-Method`), 额外的请求头(`Access-Control-Request-Headers`).

如果允许跨域, 服务器会返回如下示例:

```bash
HTTP/1.1 200 OK
Date: Wed, 09 Sep 2019 03:16:58 GMT
Server: nginx
Access-Control-Allow-Origin: http://test.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Authorization
```

如果不允许跨域, 服务器的返回头中不会有 `Access-Control-*`, 浏览器同样会报如上的 `CORS policy`问题.

#### 内容请求

只有预检请求通过浏览器才会发实际的内容请求, 也同样会加上 `Origin` 请求头, 服务器会正常返回, 并同时带上 `Access-Control-Allow-Origin` 响应头.

## 参考链接

* [浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
* [CORS MDN](https://developer.mozilla.org/zh-CN/docs/Glossary/CORS)
* [CORS跨域原理解析](https://juejin.im/post/6844903859068862472)
* [跨域资源共享 CORS 详解](https://www.ruanyifeng.com/blog/2016/04/cors.html)