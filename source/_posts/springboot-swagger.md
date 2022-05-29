---
title: springboot配置swagger以及swagger2/3之间的区别
date: 2022-05-25 11:58:31
tags: 技术杂谈
---

## 依赖区别

swagger2需要引入如下两个依赖项：

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

但是到了swagger3中只需要引入一个依赖即可：

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

## swagger配置区别

swagger2的配置代码如下：

```java
package com.example.springboot_demo.util;

import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2 //与swagger3不同的地方，swagger3使用@EnableOpenApi 
public class Swagger2Config{

    @Value("${swagger2.enabled}")
    private boolean swagger2Enabled;//读取配置，控制swagger是否启用

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2) //与swagger3不同的地方，swagger3使用OAS_30
                .apiInfo(apiInfo())
                .enable(swagger2Enabled)
                .groupName("SwaggerGroupOneApi")
                .select()
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("牛逼工具平台API接口文档")
                .termsOfServiceUrl("https://likeshan168.github.io")
                .contact(new Contact("Sherman", "https://likeshan168.github.io", "likeshan168@163.com"))
                .version("1.0")
                .description("系统API描述")
                .build();
    }
}

```

swagger2使用@EnableSwagger2进行注解，还有DocumentationType是SWAGGER_2

swagger3的配置代码：

```java
package com.example.springmvc.util;

import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

@Configuration
@EnableOpenApi //与swagger2不一样的地方，swagger2使用@EnableSwagger2
public class Swagger3Config  {

    /**
     * 读取配置
     */
    @Value("${swagger.enabled}")
    Boolean swaggerEnabled;

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.OAS_30)//与swagger2不一样的地方，swagger2使用SWAGGER_2
                .apiInfo(apiInfo())
                .enable(swaggerEnabled)
                .groupName("SwaggerGroupOneApi")
                .select()
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("牛逼工具平台API接口文档")
                .termsOfServiceUrl("https://likeshan168.github.io")
                .contact(new Contact("Sherman", "https://likeshan168.github.io", "likeshan168@163.com"))
                .version("1.0")
                .description("系统API描述")
                .build();
    }
}

```



## 访问地址的区别

swagger2的访问地址为：http://ip:port/swagger-ui.html

swagger3的访问地址为：http://ip:port/swagger-ui/ 或者 http://ip:port/swagger-ui/index.html

## 其他问题

在配置swagger的过程，碰到了404的情况，也是网上一通寻找答案，未能解决我的问题，主要的原因在于我下面这段代码：

```java
package com.example.springboot_demo;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan(basePackages = {"com.sherman.common"})
@Slf4j
public class SpringbootDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootDemoApplication.class, args);
    }
}

```

由于，我引入了另一个common的模块，所以我就通过@ComponentScan注解指定了需要扫描的包名称，但是却没有添加本项目的包在里面，导致发现不了本项目中的componet，service，controller等bean，只需要作如下修改即可：

```java
@ComponentScan(basePackages = {"com.sherman.common","com.example.springboot_demo"})
```

至此，问题得以解决。
