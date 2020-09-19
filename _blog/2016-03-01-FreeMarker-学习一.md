---
layout: article
title: FreeMarker 学习一
tags: [freemarker,Java]
key: b736f6bd-5227-4b3f-b817-676305bdd596
---

FreeMarker  是一个模板引擎, 基于模板操作, 用来生成输出文本（HTML, java 源文件等）的通用工具, 主要是用来生成HTML网页.对 FreeMarker 来说, 输出 = 模板 + 数据模型.模板就是页面大致布局, 通常是静态的文本文件, 数据就是框架中要填充的内容, 通常在 Java 程序中动态指定, 它们的关系类似于 HTML 中的标签与标签中的内容.FreeMarker 成功的将页面的显示与其背后的逻辑分离.按照顺序一般先学习 FreeMarker 的模板开发, 再学习模板背后的的程序开发.

<!--more-->

## 准备

* Maven 3
* JDK 1.7
* FreeMarker 2.3.9  

## HelloWorld 项目结构

整个项目的文件结构:

```bash
freemarker-demo
    |————/src
            |————/main
                    |————/java  
                            |——————com/albert/ch1
                                        |——————————HelloWorld.java
                    |————/resources
                            |——————com/albert/ch1
                                        |——————————HelloWorld.ftl
    |————pom.xml
```

## 添加项目依赖

FreeMarkder 是纯 Java 实现, 使用时只需要导入其 jar 包, 这里我使用 Maven 引入此 jar包, 项目的 pom.xml 里的依赖如下:

```xml
    <dependencies>
        <dependency>
            <groupId>freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.9</version>
        </dependency>
    </dependencies>
```

这样就可以使用 FreeMarker 提供的一系列类操作模板, 输出最后的结果.

## 编写模板文件

`HelloWorld.ftl`模板文件实际上是一个文本文件.模板里一些是写死的内容, 会输出到最终文件, 一些是 FreeMarkder 指令, 比如这里的`${replaceMe}`, FreeMarker 会解析指令, 然后动态生成新的内容, 最终文件就是由此两者构成.

这里 `${replaceMe}` 指令的意思是使用数据模型中的 `replaceMe` 变量值替换文本中"${replaceMe}"字符串.这里模板文件放在 src/resources/com/albert/ch1 目录下.

```html
<html>
<head>
    <title>Welcome!</title>
</head>
<body>
<p>${replaceMe}</p>
</body>
</html>
```

## 编写模板解析程序

在 src/main/java/com/albert/ch1 目录下有模板处理程序文件 `HelloWorld.java`,包名为 `com.albert.ch1`.

```java
package com.albert.ch1;
import freemarker.template.Configuration;
import freemarker.template.Template;
import java.io.File;
import java.io.OutputStreamWriter;
import java.util.HashMap;
import java.util.Map;
public class HelloWorld {
    public static void main(String[] args) throws Exception{
        //获取模板文件所在文件夹路径
        String path = HelloWorld.class.getResource("").getPath();
        File templateLocation = new File(path);
        //检查路径是否正确
        if(!templateLocation.isDirectory()) return;
        //应用级的公共配置信息, 多个模板可共享同一个Configuration
        Configuration cfg = new Configuration();
        //设置模板文件的目录
        cfg.setDirectoryForTemplateLoading(templateLocation);
        //获取模板
        Template template = cfg.getTemplate("HelloWorld.ftl");
        //建立数据模型
        Map<String,Object> root = new HashMap<String, Object>();
        root.put("replaceMe","Hello World");
        //输出到控制台
        OutputStreamWriter writer = new OutputStreamWriter(System.out);
        template.process(root,writer);

    }
}
```

## 输出结果

因为是使用 Maven 引入的相关 jar包, 所以直接使用 java 命令执行 `HelloWorld.class`的 main 方法会提示找不到用到的class错误, 不过 Maven 有插件可以帮助我们将 main 方法的执行绑定到特定的生命周期.

在 pom.xml 里添加如下插件:

```xml
<plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.1.1</version>
                <executions>
                    <execution>
                        <phase>test</phase>
                        <goals>
                            <goal>java</goal>
                        </goals>
                        <configuration>
                            <mainClass>com.albert.ch1.HelloWorld</mainClass>
                            <arguments>
                                <argument>arg0</argument>
                                <argument>arg1</argument>
                            </arguments>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

插件将在生命周期的 test 阶段执行 `com.albert.ch1.HelloWorld` 的main方法

在项目根目录上执行命令:

```bash
mvn test
```

运行结果:

```bash
[INFO]
[INFO] --- exec-maven-plugin:1.1.1:java (default) @ freemarker-demo ---
<html>
<head>
    <title>Welcome!</title>
</head>
<body>
<p>Hello World</p>
</body>
</html>[INFO] ------------------------------------------------------------
------
[INFO] BUILD SUCCESS
[INFO] -------------------------------------------------------------------
[INFO] Total time: 0.776 s
[INFO] Finished at: 2015-05-21T08:51:35+08:00
[INFO] Final Memory: 10M/155M
[INFO] -------------------------------------------------------------------
```

可以看到`<p>`与`</p>`之间的`${replaceMe}`已经替换成的`Hello World`
