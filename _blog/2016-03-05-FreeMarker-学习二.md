---
layout: article
title: 2016-01-01 FreeMarker 学习二
tags: [freemarker,Java]
key: dbc75086-587d-400b-b02d-7ecd3af371c2
---

上一节讲了一个非常简单的入门级 HelloWorld 示例, 这一节讲对 FreeMarker 非常重要的两个东西: 数据模型、模板指令. 

<!--more-->

## 数据模型

FreeMarker 中的数据基本结构整体上看是树型的, 细节上看都是键值对. 

最基本的键值对是标量, 只能存储单一的值, 有如下四种标量:

* 字符串
* 数字
* 时间
* 布尔值

标量可以直接通过名称获取其存放的值. 

多个标量可以放在容器中, 容器也可以添加到其他容器中. 容器分为两种: 哈希表与序列. 哈希表通过名称查找其值, 典型的key-value形式, 序列通过序号查找值, 也可以说其名称为序列名加所在序号. (在程序中实现时容器也可以使用java的集合框架中类, 也可以使用java bean, 此时等效于哈希表)

获取变量需要从树的根节点root开始(root是隐含的不用写), 每级通过`.`分隔. 

比如如下的数据模型:

```bash
(root)
    |
    +————fruits
            |
            +————apple
                    |
                    +—————color = "red"
                    |
                    +—————price = 30
            |
            +————orange
                    |
                    +——————color = "orange"
                    |
                    +——————price = 40

    |————persons
            |
            +————(1st)
                    |
                    +——————name = "Andy"
                    |
                    +——————age = 31
            |
            +—————(2nd)
                    |
                    +——————name = "Tom"
                    |
                    +——————age = 44
```

这里fruits就是一个哈希表, 要获取苹果的价钱, 可以使用 `fruits.apple.price`. 
persons 是一个序列, 获取第一个人的名字, 可以使用 `persons[0].name`, 注意序列是从0开始计数. 

在获取变量值时如果变量不存在（值为null也相当于不存在）, 可以设置一个默认值

```bash
${fruits.apple.price!33}
```

可以使用两个??判断变量是否存在

```bash
<#if user??>${user}</#if>
```

## 模板指令

一个模板一般包含如下4部分:

* 文本, 直接输出到最终结果中
* 插值, 使用`${..}`的格式
* 注释, 使用<#-- 注释内容 -->格式编写的freemarker专用注释
* 指令, 和HTML标签类似,使用`<#tag></#tag>`格式或者`<@tag></@tag>`

这里重点讲freemarker的自带指令:

### if

基本格式是

```bash
<#if condition1>condition1为真时有效
<#elseif condition2>conditon1为假且condition2为真时有效
<#else>condition1为假, condition2为假时有效
</#if>
```

`<#elseif>`、`<#else>`标签可以不带. conditon 必须是一个能判断 false 与 true的表达式或者布尔类型的值, 如果不是会抛出异常:

```bash
freemarker.core.NonBooleanException: Error on line 8, column 10 in HelloWorld.ftl
Expecting a boolean (true/false) expression here
Expression output does not evaluate to true/false
it is an instance of freemarker.template.SimpleScalar
 at freemarker.core.Expression.isTrue(Expression.java:150)
```

比如如下示例, 如果有output变量, 并且其值为布尔类型的true时才输出`It's true`, 否则输出`It's false.`

```bash
<#if output == true>
   It's true.
<#else>
    It's false
</#if>
```

### switch

基本格式是:

```bash
<#switch value>
<#case refValue>...<#break>
<#case refValue>...<#break>
<#default>
</#switch>
```

### list指令

`<#list sequence as item></#list>`可以用来遍历容器中的内容, 比如:

```bash
    <#list persons as person>
        ${person.name}:${person.age}
    </#list>
```

使用到的类 Person

```java
public static class Person{
        String name;
        Integer age;
        public Person(String name, Integer age){
            this.name = name;
            this.age = age;
        }
        //..getter()
        //..setter()
    }
```

对应的数据模型实现:

```java
    List<Person> persons = new ArrayList<Person>();
    persons.add(new Person("Andy",31));
    persons.add(new Person("Tom", 44));
    root.put("persons", persons);
```

在遍历中可以使用指令`<#break>`跳出迭代. 

### include指令

可以使用`<#include filename [options]>`指令来引入其他模板文件. 比如之前的 HelloWorld 项目, 在 HelloWorld.ftl 同目录下建立 footer.ftl, 内容为:

```bash
<p>
    I'm footer.
</p>
```

这样在 HelloWorld.ftl 中就可以使用如下方式引用:

```bash
<#include "footer.ftl">
```

options代表可选参数:encoding与parsem, encoding指定对应文件的编码, parse指定是否需要解析文件中的指令, 默认是true. 

### assign

可以使用`<#assign name="value">`的形式在模板文件中自定义变量,模板其他地方就可以通过`${name}`使用. 

### noparse

使用`<#noparse>...</#noparse>`指定不解析标签内包含的其他指令. 

### import

import指令可以用来导入其他模板的所有变量并放入指定的哈希容器, 比如:

```bash
<#import "/lib/common.ftl" as com>
```

这样就可以在本模板中使用 common.ftl 中定义的变量, 而且不用担心两模板中的同名变量冲突, 因为有不同的命名空间. 

### setting

该指令`<#setting name=value>`用于设置freemarker的运行环境, name可选值有:

* local: 设置模板所用的国家/语言
* number_format: 指定数字的输出格式
* boolean_format: 指定布尔值的输出格式
* date_format: 指定时间的输出格式
* time_zone: 指定输出时间所在的时区
value在之后的数值和类型中写. 

### 其他

还有其他macro nested return指令等
