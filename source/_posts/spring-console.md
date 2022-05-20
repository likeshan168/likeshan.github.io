---
title: java控制台程序添加springboot支持
date: 2022-05-15 23:45:57
tags: 技术杂谈
---

## 创建项目

首先创建一个普通的maven项目，方式有很多种，这里就不再详细阐述，我这里通过命令行的方式已经创建了一个maven项目，添加依赖，可以通过两种方式：

1. 直接添加  `spring-boot-starter-parent` 这个作为parent节点如下所示：

   ```xml
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>2.6.7</version>
   </parent>
   ```

   然后添加`spring-boot-starter`的依赖

   ```xml
   <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter</artifactId>
   </dependency>
   ```

   因为上面parent指定了版本，可以在依赖中直接继承上面的版本号

2. 不用`spring-boot-starter-parent` 这个，直接添加spring-boot-starter依赖，并指定版本号

   ```xml
   <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter</artifactId>
         <version>2.6.7</version>
   </dependency>
   ```

   当然还需要添加一个插件

   ```xml
   <plugin>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-maven-plugin</artifactId>
             <version>2.6.7</version>
             <executions>
               <execution>
                 <goals>
                   <goal>repackage</goal>
                 </goals>
               </execution>
             </executions>
   </plugin>
   ```

## 实现接口

添加完了依赖之后，我们需要CommandLineRunner接口，并重写run方法，代码如下

```java
package com.sherman;

import com.sherman.service.HelloWorldService;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Hello world!
 *
 */
@SpringBootApplication
public class App implements CommandLineRunner
{

    @Autowired
    private HelloWorldService helloWorldService;

    public static void main( String[] args )
    {
        SpringApplication.run(App.class, args);
        System.out.println( "Hello World!" );
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println("程序的真正入口地址");
        helloWorldService.sayHello();
    }
}
```

接下来，我们定义一个服务

```java
package com.sherman.service;

 /**
  * HelloWorldService
  */
 public interface HelloWorldService {
     void sayHello();
 }
```

```java
package com.sherman.service.impl;

import com.sherman.service.HelloWorldService;

import org.springframework.stereotype.Service;

/**
 * HelloWorldServiceImpl
 */
@Service
public class HelloWorldServiceImpl implements HelloWorldService {

    @Override
    public void sayHello() {
        System.out.println("hello world");
    }
}
```

这样我们就可以在控制台程序中使用springboot的功能，进行配置，依赖注入等强悍的功能，非常方便，大家也来试试吧

## 题外话

再多说一点有关`spring-boot-starter-parent` 的知识，我们可以看到，引入的时候是通过parent节点进行引入的，说明`spring-boot-starter-parent`是作为父项目进行引入，这样当前的项目就可以继承`spring-boot-starter-parent`相关的配置，我们可以进入到`spring-boot-starter-parent`定义中去，查看具体的配置内容，截取部分内容看看：

```xml
<modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.6.7</version>
  </parent>
  <artifactId>spring-boot-starter-parent</artifactId>
  <packaging>pom</packaging>
  <name>spring-boot-starter-parent</name>
  <description>Parent pom providing dependency and plugin management for applications built with Maven</description>
  <properties>
    <java.version>1.8</java.version>
    <resource.delimiter>@</resource.delimiter>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>
```

看几个关键的东西 `<packaging>pom</packaging>` 这个是指打包的类型为pom，父类型就必须指定为pom，默认的是jar，作为内部调用或者后台服务使用使用jar类型，如果是在tomcat或者jetty等容器中运行可以指定为war类型，当然还有其他的类型（如：maven-plugin、ejb、ear、par、rar等）。

还可以看到

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.6.7</version>
 </parent>
```

`spring-boot-starter-parent`的父项目为`spring-boot-dependencies`，再进入到`spring-boot-dependencies` 定义中去，首先`spring-boot-dependencies`的打包类型也是pom，还有一个比较关键的是`dependencyManagement`：

```xml
<dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-amqp</artifactId>
        <version>${activemq.version}</version>
      </dependency>
      <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-blueprint</artifactId>
        <version>${activemq.version}</version>
      </dependency>
      <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-broker</artifactId>
        <version>${activemq.version}</version>
      </dependency>
        ......
```

截取了部分内容，`spring-boot-dependencies`内部使用了 `dependencyManagement`进行依赖的管理，那`dependencyManagement`与`dependencies`的区别在哪儿呢？dependencyManagement包裹的依赖，相当于定义一个可以继承的依赖，但不是必须的，也就是说，子项目要想使用其中定义的依赖，需要在dependencies标签内部显示指定（groupId、artifactId）,其中version可以不指定，如果不指定，默认就继承父项目定义的version，如果指定了就覆盖父项目中的版本，使用本地定义的依赖，而父级的是不会引入的。而如果父项目中只是包含了dependencies，那么子项目就会继承所有在dependencies定义的依赖，而子项目也不用再指定具体的依赖。dependencyManagement主要用于统一管理版本。
