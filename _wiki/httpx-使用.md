---
layout: article
title: httpx 使用
tags: []
key: e25121ab-d128-416a-8630-310676a082e3
---

httpx 支持同步和异步的 http 请求, 是 requests 的替代品, 需要 Python 3 以上.

<!--more-->

## 安装

```bash
pip install httpx
```

### 示例

同步请求:

```python
import httpx

r = httpx.get('https://www.example.org/')
print(r.text)
```

异步请求:

```python
import httpx
import asyncio

async def main():
    async with httpx.AsyncClient() as client:
        r = await client.get('https://www.example.org/')
        print(r.text)

asyncio.run(main())
```
