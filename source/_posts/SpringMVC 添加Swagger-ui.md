---
title: SpringMVC配置Swagger构建Api文档
description: Swagger 是一款RESTFUL接口的文档在线自动生成+功能测试功能软件。Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。总体目标是使客户端和文件系统作为服务器以同样的速度来更新。文件的方法，参数和模型紧密集成到服务器端的代码，允许API来始终保持同步。Swagger 让部署管理和使用功能强大的API从未如此简单。
date: 2017-07-25 21:43:00
comments: true
tags: 
    - Spring  
categories:
    - Spring
---

# 版本描述
Spring4.+, springfox-swagger2 2.6

# SpringMVC整合Swagger-ui 

## 添加jar
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.6.0</version>
</dependency>
<!-- swagger-ui 为项目提供api展示及测试的界面 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

## 配置Swagger
```java
@Configuration
@EnableSwagger2
@ComponentScan(basePackages = "com.bmkit.controller")
public class SwaggerConfig {
    @Bean
    public Docket getDocket(){

        Docket docket=new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .paths(regex("/marker/.*"))
                .build()
                .pathMapping("/");
        return docket;
    }

    public ApiInfo apiInfo(){
        ApiInfo apiInfo=new ApiInfo(
                "SpringMVC整合Swagger-ui API",
                "SpringMVC整合Swagger-ui API",
                "1.1",
                "",
                contact(),
                "",
                "");
        return apiInfo;
    }

    public Contact contact(){
        Contact contact=new Contact("yuanwj","","yuanwj@gmail.com");
        return contact;
    }
}


@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html").addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
}
```

## 访问

http://localhost:port/swagger-ui.html

## 效果
![SpringMVV+Swagger-ui效果图][SpringMVV+Swagger-ui]

[SpringMVV+Swagger-ui]: /images/mvc+swagger.jpg
