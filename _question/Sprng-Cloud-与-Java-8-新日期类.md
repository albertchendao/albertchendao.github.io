---
layout: article
title: Spring Cloud 与 Java 8 新日期类
tags: []
key: f81cacec-b48e-4387-b262-bd606343f9aa
---

Spring Cloud 中使用 Java 8 时间日期 API（LocalDate等）的序列化问题。

<!--more-->

## 问题

Spring Cloud 默认序列化 Java 8 的日期会变成数值，比如: `[2018,9,10]`, 然后如果使用 `Feign` 进行反序列化就会出现:

```
JSON parse error: Can not construct instance of java.time.LocalDate: no suitable constructor found, can not deserialize from Object value
```

## 解决方法

在项目中引入专门的序列化包:

```
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
</dependency>
```

然后注入到全局的 `ObjectMapper`:

```java
@Bean
public ObjectMapper serializingObjectMapper() {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    objectMapper.registerModule(new JavaTimeModule());
    return objectMapper;
}
```


