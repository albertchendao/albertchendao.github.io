---
layout: article
title: Maven 手动安装依赖包
tags: []
key: 709096ba-811a-4eae-b2ec-f603c0fed03d
---

今天遇到一个问题，远程 Maven 中央仓库没有 `javax.servlet:javax.servlet-api:2.5`，配置另一个远程仓库又太小题大做，最好直接安装到本地。但是直接复制对应的包到本地仓库是不行的，必须有对应的 `pom.xml` 文件，这种时候就需要进行手动安装，具体步骤如下。

<!--more-->

## 下载对应的 jar 包

从网上找到对应的 jar 包，下载到本地。

## 安装到本地仓库

在命令行执行如下命令：

```
mvn install:install-file -DgroupId=java.servlet -DartifactId=javax.servlet-api -Dversion=2.5 -Dpackaging -Dfile=D:/SERVLET-API.JAR
```

参数解释:

> -DgroupId=包名  
> -DartifactId=项目名  
> -Dversion=版本号  
> -Dpackaging=jar  
> -Dfile=jar文件所在路径

## 项目中引用

在项目的 `pom.xml` 中加入如下依赖：

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>  
```