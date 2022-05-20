---
title: 普通Maven项目中添加springboot支持
date: 2022-05-13 23:44:04
tags: 技术杂谈
---

## 创建项目

创建springboot项目有很多种方式，可以通过IDE（idea、eclipse等）工具，或者[spring initializr](https://start.spring.io/)，但是本文的重点是通过创建一个普通的maven项目，然后通过添加springboot的相关依赖去构建springboot项目，主要是为了让自己对springboot项目有一个大致的了解。创建maven项目可以参考我之前的文章，[通过maven命令创建项目](https://www.toutiao.com/article/7094642417542087176/?log_from=631fee930fa56_1652456805449)，此处不再详细描述

## 添加依赖

在pom.xml文件加入以下内容

```xml
 <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.7</version>
  </parent>
```

这就是Spring Boot的父级依赖，加入之后项目就变成了Spring Boot项目。`spring-boot-starter-parent`是一个特殊的starter，它用来提供相关的maven默认依赖。之后再引入。其他的依赖时，可以不用指定version标签。接下我们引入web应用相关的依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 测试验证

修改main方法，加入@SpringBootApplication注解

```java
package com.sherman.demo.SpringBoot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootDmeo {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootDmeo.class, args);
    }
}
```

添加一个controller，实现一个简单的api接口

```java
package com.sherman.demo.SpringBoot.Controllers;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorldController {
 
    @RequestMapping("/")
    public String sayHello(){
        return "Hello World";
    }
}
```

将程序运行起来，在浏览器中输入localhost:8080(默认端口：8080)，可以看到输出接口：Hello World，至此，springboot项目已经成功运行起来了。
