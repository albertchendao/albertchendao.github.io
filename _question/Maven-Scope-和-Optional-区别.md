---
layout: article
title: Maven Scope 和 Optional 区别
tags: []
key: 0b8717f9-e7a6-49ff-9b25-3ef9b635af09
---

Maven 的 `pom.xml` 文件中配置依赖包有两个参数: `scope`、`optional`，两个有区别的。

<!--more-->

## Scope

`scope` 控制的是依赖包是否加入本项目的 `classpath`：


| Scope    | 编译 classpath | 测试 classpath | 运行时 classpath | 传递性 |
| ---      |   --- |   --- |    --- |   --- |
| compile  |    Y  |   Y   |   Y    |   Y   |
| test     |    -  |   Y   |   -    |   -   |
| provided |    Y  |   Y   |   -    |   -   |
| runtime  |    -  |   Y   |   Y    |   Y   |
| system   |    Y  |   Y   |   -    |   Y   |

## Optional

`optional` 控制的是依赖包的传递性。

比如项目 A 依赖项目 B，项目 B 依赖项目 C，如果在项目 B 的 pom.xml 中配置:

```xml
<dependency>
    <groupId>C</groupId>
    <artifactId>C</artifactId>
    <version>1.0</version>
    <optional>true</optional>
</dependency>
```

这样项目 A 中就不会自动将 C 引入进来，需要手动添加。