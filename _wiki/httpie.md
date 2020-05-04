---
layout: article
title: httpie
tags: [Wiki]
key: 
---

httpie 是一个类似于 curl 用于 HTTP 命令行工具，但是对人更友好。

## 安装

1. MAC: `brew install httpie`
2. Linux: `pip install --user httpie`

安装成功后直接输入命令检测: `http`

## 用法

```bash
# 显示返回头、返回体
http ipip.net
# 显示请求头、返回头、返回体
http -v ipip.net
# 显示返回头
http -h ipip.net
# 显示返回体
http -b ipip.net
# 下载文件
http -d ipip.net
# URL 参数(使用 ==)
http -f POST ipip.net username=='me'
# 提交表单
http -f POST ipip.net username='me'
# 表单上传
http -f POST example.com/jobs username='me' file@~/test.pdf
# DELETE 请求
http DELETE ipip.net
# 提交 JSON (默认)
http PUT ipip.net username='me' password='pwd'
# 提交 JSON (非字符串使用 :=)
http PUT ipip.net username='me' password='pwd' age:=28 a:=true streets:='["a", "b"]'
# 提交 JSON
http PUT ipip.net < info.json
# 自定义请求头(使用 : )
http -v ipip.net  User-Agent:mimvp-agent/1.0 'Cookie:a=b;b=c'  Referer:http://ipip.net/

# 认证
http -a username:password ipip.net
http --auth-type=digest -a username:password ipip.net

# HTTP 代理
http --proxy=http:http://217.107.197.174:8081 ipip.net
http --proxy=http:http://user:pass@217.107.197.174:8081 ipip.net
http --proxy=https:http://112.114.96.34:8118 ipip.net
http --proxy=https:http://user:pass@112.114.96.34:8118 ipip.net
```
