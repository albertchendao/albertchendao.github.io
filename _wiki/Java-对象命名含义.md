---
layout: article
title: Java 对象命名含义
tags: [Wiki]
key: 
---

Java 中各种 xxxO 的含义。

## 分类

* DO（ Data Object）：与数据库表结构一一对应，通过DAO层向上传输数据源对象(备注: 也有人称为 PO 等)。
* DTO（ Data Transfer Object）：数据传输对象，Service或Manager向外传输的对象。
* BO（ Business Object）：业务对象。 由Service层输出的封装业务逻辑的对象，可包含多个其他对象。
* AO（ Application Object）：应用对象。 在Web层与Service层之间抽象的复用对象模型，极为贴近展示层，复用度不高。
* VO（ View Object）：显示层对象，通常是Web向模板渲染引擎层传输的对象。
* POJO（ Plain Ordinary Java Object）：在本手册中， POJO专指只有setter/getter/toString的简单类，包括DO/DTO/BO/VO等。
* Query：数据查询对象，各层接收上层的查询请求。 注意超过2个参数的查询封装，禁止使用Map类来传输。

## 命名规则

* 数据对象：xxxDO，xxx即为数据表名。
* 数据传输对象：xxxDTO，xxx为业务领域相关的名称。
* 展示对象：xxxVO，xxx一般为网页名称。
* POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO。

## 参考链接

* [阿里巴巴Java开发手册](https://github.com/alibaba/p3c/blob/master/%E9%98%BF%E9%87%8C%E5%B7%B4%E5%B7%B4Java%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C%EF%BC%88%E8%AF%A6%E5%B0%BD%E7%89%88%EF%BC%89.pdf)
