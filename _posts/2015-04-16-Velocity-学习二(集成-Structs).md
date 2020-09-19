---
layout: article
title: Velocity 学习二(集成 Structs)
tags: [velocity,Java]
key: f458395e-93ec-42e3-8e95-498d7701a6c4
---

`Struts2`默认已经支持`velocity`视图展示, 因此在配置时方便了很多, 几乎与先前使用`jsp`的配置没两样.

<!--more-->

## 准备

* Maven 3
* JDK 1.7
* Volecity
* Eclipse

## 项目搭建

### 创建新项目

复制项目 `velocity_0100_helloworld`, 修改项目名为 `velocity_0200_struts2`.

### 引入依赖

修改`pom.xml` 引入`struts2`包与`servlet`包

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.admin.velocity</groupId>
  <artifactId>velocity_0200_struts2</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>velocity_0200_struts2 Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.3</version>
      <scope>test</scope>
    </dependency>
   <dependency>  
         <groupId>org.apache.struts</groupId>  
            <artifactId>struts2-core</artifactId>  
            <version>2.3.1.2</version>  
        </dependency>  
        <dependency>
	    <groupId>org.apache.velocity</groupId>
	    <artifactId>velocity</artifactId>
	    <version>1.7</version>
	</dependency>
	<dependency>
	    <groupId>org.apache.velocity</groupId>
	    <artifactId>velocity-tools</artifactId>
	    <version>2.0</version>
	</dependency>
        <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
  <build>
    <finalName>hellostruts2</finalName>
  </build>
</project>
```
### 配置 `web.xml` 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>  
 <web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee   
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">  
    <filter>  
        <filter-name>Struts2</filter-name>  
        <filter-class>  
            org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>  
        <init-param>  
            <param-name>struts.multipart.saveDir</param-name>  
            <param-value>/tmp</param-value>  
        </init-param>  
    </filter>  
    <filter-mapping>  
        <filter-name>Struts2</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
   
    <welcome-file-list>  
        <welcome-file>index.jsp</welcome-file>  
    </welcome-file-list>  
 </web-app>  
```

`Struts2`中的`struts.multipart.saveDir`主要是用来设置上传文件的临时目录

* 配置一: 不设置`struts.multipart.saveDir`, 这种情况下, 临时文件就放在tomcat安装目录下的`work\Catalina\localhost\项目名称`目录下.
* 配置二: `<constant name="struts.multipart.saveDir" value="/tempfile"/>`, 这种情况下, 临时文件放在项目所在的根磁盘下的`tempfile`目录下. 如项目放在D盘, 则该tempfile临时文件夹就在D盘根目录下.
* 配置三: `<constant name="struts.multipart.saveDir" value="tempfile"/>`这种情况比上面少了一个斜杠, 这种情况下, 临时文件放在项目所在的`tomcat`的`bin`目录下的`tempfile`目录下.
如项目放在`D:\tomcat\webapps`目录, 则该`tempfile`临时文件夹就在`D:\tomcat\bin`目录下.
### 建立`struts.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
 <!DOCTYPE struts PUBLIC  
     "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"  
     "http://struts.apache.org/dtds/struts-2.3.dtd">  
 <struts>  
    <package name="demo" extends="struts-default">  
        <action name="getname" class="com.admin.action.UserAction">  
            <result type="velocity">/templates/welcome.vm</result>  
        </action>  
    </package>  
 </struts>  
```

### 建立 Action 类

```java
package com.admin.action;
public class UserAction {
	
	private String username;  
	  
    public String getUsername() {  
        return username;  
    }  
  
    public void setUsername(String username) {  
        this.username = username;  
    }  
      
    public  String execute(){  
        return "success";  
    }  
}
```

### 建立 `vm` 文件

根据 `struts.xml` 里配置的路径在`webapp`下建立`templates`文件夹,再在其下建立 `welcome.vm`

```html
<html>  
 <head>  
 <title>Struts2 Velocity</title>  
 </head>  
 <body>  
    Hello, $username 
 </body>  
 </html>
```

然后打包部署即可.
