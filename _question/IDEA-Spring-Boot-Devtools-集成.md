---
layout: article
title: IDEA Spring Boot Devtools 集成
tags: []
key: 8029d9a3-37ac-4a7f-8b3f-87723dce38cd
---

IDEA 中使用 `spring-boot-devtools` 进行热部署, 有代码变动自动重启.

<!--more-->

## Maven 配置

`pom.xml` 中加入如下配置:

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
            <scope>runtime</scope>
        </dependency>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.1.RELEASE</version>
                <configuration>
                    <fork>true</fork>
                    <executable>true</executable>
                    <mainClass>${main.class}</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

```

## IDEA 配置

* IDEA 设置 `setting -> Build,Execution,Deployment -> Compiler` 中开启 `Build Project Automatically`
* 通过 `ctrl+shift+alt+/` 打开 `Registry`, 开启 `auto-making when app running`
* 然后重启 IDEA
