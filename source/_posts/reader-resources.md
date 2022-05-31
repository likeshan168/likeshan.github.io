---
title: 几种获取resources目录下的文件方式
date: 2022-05-29 22:57:53
tags: 技术杂谈
---

## 前言

一般我们的配置信息默认都是会配置在/src/main/resources/application.properties(或者application.yml)文件中，当然，也可以在resources文件夹下添加自己的配置文件，甚至子目录中添加自己的配置文件，那么我们又该如何读取自己添加的配置文件中的内容呢？

## 准备

我们先定义一个公共的输出配置信息的方法如下：

```java
private static void getProperties(InputStream inputStream) {
        Properties properties = new Properties();
        if (inputStream == null) {
            return;
        }
        try {
            properties.load(inputStream);
            properties.list(System.out);
        } catch (Exception ex) {
            ex.printStackTrace();
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

这里是通过java.util下的Properties类来获取配置文件中的属性

添加自定义的配置文件，在resources目录下添加子目录config并添加配置文件db.properties

![image-20220530225539228](reader-resources/image-20220530225539228.png)

内容如下：

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
```

在java中，resources文件夹下的文件在编译后，都是为根目录（classpath）。接下来，准备采用以下的6种方式进行配置内容的读取

## 六种方式

### 方法一

```java
URL path = this.getClass().getClassLoader().getResource("config/db.properties"); // 注意路径不带/开头
getProperties(path.openStream());
```

### 方法二

```java
URL path = this.getClass().getResource("/config/db.properties"); //路径需要以/开头
getProperties(path.openStream());
```

### 方法三

在springboot项目我还可以使用如下的方式：

```java
InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream("config/db.properties");//与方法一类似，只不过直接返回了InputStream类型
getProperties(inputStream);
```

### 方法四

springboot项目中使用

```java
inputStream = this.getClass().getResourceAsStream("/config/db.properties");//与方法二类似，只不过返回了InputStream类型了
getProperties(inputStream);
```

### 方法五

springboot项目中使用

```java
ClassPathResource classPathResource = new ClassPathResource("config/db.properties");
getProperties(classPathResource.getInputStream());
```

### 方法六

springboot项目中使用，通过@Value注解，但是我们还需要通过@PropertySource("classpath:config/db.properties")

注解指定配置文件的路径，如果是默认的配置文件，如：application.properties(.yml)就不需要指定路径

```java
@SpringBootApplication
@PropertySource("classpath:config/db.properties")
public class App implements CommandLineRunner{...}
```



```java
 @Value("${jdbc.driver}")
 private String driver;
```

通过上述6种方法都可以成功获取到自定义配置文件中的配置信息，如果大家还有更好的方式，可以评论区留言。
