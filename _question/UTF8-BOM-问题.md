---
layout: article
title: UTF8 BOM 问题
tags: []
key: a5c6d987-f025-4515-9ba0-31d81e4756ff
---

Excel 直接打开无 BOM 的 UTF-8 CSV 文件时会乱码.

<!--more-->

## 问题

Excel 直接打开无 BOM 的 UTF-8 CSV 文件时会乱码.

## JAVA 添加 BOM

```java
    public static final byte[] BOM_BYTES = new byte[]{(byte) 0xEF, (byte) 0xBB, (byte) 0xBF};
    public static final String BOM_STR = new String(BOM_BYTES);

    File file = null;
    OutputStreamWriter out = new OutputStreamWriter(new FileOutputStream(file));
```

## Python 添加 BOM

```python
import codecs

if __name__ == "__main__":
    with open('./tmp/1.csv', 'wb') as w:
        w.write(codecs.BOM_UTF8)
        with open('/Users/albert/Downloads/应用日数据.csv', 'r', encoding='utf-8') as r:
            for l in r.readlines():
                w.write(l.encode('utf-8'))
```
