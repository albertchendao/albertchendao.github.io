---
layout: article
title: ElasticSearch 问题
tags: []
key: 9933302a-e575-45c3-bf4e-19350cead69b
---

## NoClassDefFoundError

编写了一个  `elastic-spring-boot-starter` 通过如下 `pom.xml`来集成 ES 6.8.11

```xml
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <org.mapstruct.version>1.3.1.Final</org.mapstruct.version>
        <lombok.version>1.16.20</lombok.version>
        <deploy.skip>false</deploy.skip>
        <spring.boot.version>1.5.6.RELEASE</spring.boot.version>
    </properties>

	<artifactId>elastic-spring-boot-starter</artifactId>

	<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <scope>provided</scope>
        </dependency>

        <!--   ES 组件依赖 -->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-client-sniffer</artifactId>
            <version>6.8.11</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>6.8.11</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-client</artifactId>
            <version>6.8.11</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>6.8.11</version>
            <exclusions>
                <exclusion>
                    <groupId>org.elasticsearch</groupId>
                    <artifactId>elasticsearch</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

然后在其他项目里依赖:

```xml
        <dependency>
            <groupId>cn.jiguang.iapp</groupId>
            <artifactId>elastic-spring-boot-starter</artifactId>
            <version>${project.version}</version>
        </dependency>

	<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

启动的时候会报如下错误:

```bash
Caused by: java.lang.NoClassDefFoundError: org/elasticsearch/action/main/MainRequest
	at cn.jiguang.iapp.starter.elastic.EsAutoConfiguration.restHighLevelClient(EsAutoConfiguration.java:39)
	at cn.jiguang.iapp.starter.elastic.EsAutoConfiguration$$EnhancerBySpringCGLIB$$ac5481a9.CGLIB$restHighLevelClient$0(<generated>)
	at cn.jiguang.iapp.starter.elastic.EsAutoConfiguration$$EnhancerBySpringCGLIB$$ac5481a9$$FastClassBySpringCGLIB$$c3ff5bdc.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invokeSuper(MethodProxy.java:228)
	at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:358)
	at cn.jiguang.iapp.starter.elastic.EsAutoConfiguration$$EnhancerBySpringCGLIB$$ac5481a9.restHighLevelClient(<generated>)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:162)
```

解压启动的包发现实际使用的 `org.elasticsearch:elasticsearch` 是 2.4.5, 也就是 `spring-boot-dependencies` 中定义的, 这个是 Maven `dependencyManagement` 依赖传递引起的, 解决办法就是在引入 `elastic-spring-boot-starter` 的时候先强制指定 `elasticsearch` 版本:

```xml
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>6.8.11</version>
        </dependency>
        <dependency>
            <groupId>cn.jiguang.iapp</groupId>
            <artifactId>elastic-spring-boot-starter</artifactId>
            <version>${project.version}</version>
        </dependency>
```



## 参考链接

* [解決 Elasticsearch 使用 Java High Level REST Client 時出現 NoClassDefFoundError 錯誤](https://medium.com/@hsiehjenhsuan/%E8%A7%A3%E6%B1%BA-elasticsearch-%E4%BD%BF%E7%94%A8-java-high-level-rest-client-%E6%99%82%E5%87%BA%E7%8F%BE-noclassdeffounderror-%E9%8C%AF%E8%AA%A4-10077fcda6b3)
* [Maven 包冲突](https://spring.io/blog/2016/04/13/overriding-dependency-versions-with-spring-boot)
* [Spring Boot 修改组件版本](https://www.cnblogs.com/zhangjianbin/p/10076427.html)
