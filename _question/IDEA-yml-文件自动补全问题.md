---
layout: article
title: IDEA yml 文件自动补全问题
tags: []
key: c92a3e45-0f05-4c04-aaaa-7028e7ca0d94
---

用 IDEA 开发 Spring Boot 程序, `yml` 配置里不能自动补全.

<!--more-->

## 问题

用 IDEA 开发 Spring Boot 程序, 编写 `resources/application.yml`文件的时候发现有部分变量可以自动补全, 比如如果引入了 `spring-boot-starter-web`可以写: `server` 会自动提示 `port` 等参数.

查找资料是需要引入如下包: 

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

然后使用 `@ConfigurationProperties` 和 `@EnableConfigurationProperties`定义自己的属性:

```java
@Data
@ConfigurationProperties(prefix = "iapp.elasticsearch")
public class EsProperties {
    /**
     * 账号
     */
    private String username;
    /**
     * 密码
     */
    private String password;
    /**
     * IP
     */
    private List<String> ips;
    /**
     * 端口
     */
    private Integer port;
}

@Configuration
@EnableConfigurationProperties(EsProperties.class)
public class EsConfig {
}
```

IDEA 点击 `Build -> Rebuid Project` 会在项目的 `target/classes/META-INFO` 下自动生成元数据文件: `spring-configuration-metadata.json` 文件, IDEA 会使用它在编辑 `application.yml` 的时候自动补全, 但是我试了半天都没有生成 ``spring-configuration-metadata.json`` 文件.

> 注意: IDEA 中已经开启了 Enable annotation processing

## 原因

`spring-boot-configuration-processor` 的机制是通过注解在编译期间生成额外的文件.

他需要用到 `maven-compiler-plugin`插件, 这个插件默认会通过  `SPI` 查找并运行所有 jar 包下的 `services/javax.annotation.processingProcessor` 配置的执行器.

但是因为我项目中引入了 `lombok`, `mapstruct`, 所以配置的 `maven-compiler-plugin` 是这样:

```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>UTF-8</encoding>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${org.mapstruct.version}</version>
                        </path>
                        <!-- other annotation processors -->
                    </annotationProcessorPaths>
                </configuration>
                <version>3.6.2</version>
            </plugin>
```

注意这里添加了 `annotationProcessorPaths` 配置, 按照这个 `maven-compiler-plugin`的官方文档, 如果指定了 `annotationProcessorPaths`, 这个`maven-compiler-plugin` 就不会走 `SPI` 机制, 而是只运行配置的注解驱动器, 所以 `spring-boot-configuration-processor` 失效了.

## 解决方法

### 方案一(推荐)

去掉所有的 `annotationProcessorPaths`, 全部走 `SPI`, 对应的 `maven-compiler-plugin`:

```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>UTF-8</encoding>
                </configuration>
                <version>3.6.2</version>
            </plugin>
```

但是注意 `mapstruct` 推荐的方案是:

```xml
<dependencies>
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>${org.mapstruct.version}</version>
    </dependency>
</dependencies>
...
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>1.8</source> <!-- depending on your project -->
                <target>1.8</target> <!-- depending on your project -->
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${org.mapstruct.version}</version>
                    </path>
                    <!-- other annotation processors -->
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

`org.mapstruct:mapstruct` 包实际是不包含注解驱动器的, 所以要改成:

```xml
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${org.mapstruct.version}</version>
            <!--  mapstruct 在编译期生成 class, 运行时不需要自身的引用, 故不用传递  -->
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct-processor</artifactId>
            <version>${org.mapstruct.version}</version>
            <scope>provided</scope>
        </dependency>
```

### 方案二

既然指定的 `annotationProcessorPaths` 缺少了 `spring-boot-configuration-processor` 的注解驱动器, 加上就好了.

## 与 Maven Profle 集成问题

项目是通过 Maven 的 Profile 打包, `yml`文件被放在了 `resources/profiles/dev` 目录, IDEA 检测不到, 也不能自动补全.

暂时的解决办法是在 `File -> Project Structure -> Facets -> Customize Spring Boot -> + ` 手动添加 `yml` 文件.

## 参考资料

* [Configuration Metadata](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-configuration-metadata.html#configuration-metadata-format)
* [Maven Compile Plugin](https://maven.apache.org/plugins/maven-compiler-plugin/compile-mojo.html)
* [IDEA Spring properties auto completion](https://stackoverflow.com/questions/44997270/intellij-spring-properties-auto-completion-in-yml-files)
* [IDEA Spring Boot](https://www.jetbrains.com/help/idea/spring-boot.html#custom-configuration-files)
