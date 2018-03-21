---
title: SpringBoot配置Swagger构建Api文档
description: Swagger 是一款RESTFUL接口的文档在线自动生成+功能测试功能软件。Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。总体目标是使客户端和文件系统作为服务器以同样的速度来更新。文件的方法，参数和模型紧密集成到服务器端的代码，允许API来始终保持同步。Swagger 让部署管理和使用功能强大的API从未如此简单。
date: 2017-07-25 21:43:00
comments: true
tags: 
    - Spring  
    - SpringBoot
categories:
    - Spring
    - SpringBoot
---

# 描述
Swagger 是一款RESTFUL接口的文档在线自动生成+功能测试功能软件。Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。总体目标是使客户端和文件系统作为服务器以同样的速度来更新。文件的方法，参数和模型紧密集成到服务器端的代码，允许API来始终保持同步。Swagger 让部署管理和使用功能强大的API从未如此简单。

# SpringBoot整合Swagger

## 引用jar包
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.2.2</version>
</dependency>
```
## 创建Swagger配置类
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Inject
    private ConfigProperties configProperties;

    @Bean
    public Docket getDocket(){
        Docket docket=new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .paths(regex("/api/.*"))
                .build();
        return docket;
    }

    public ApiInfo apiInfo(){
        ApiInfo apiInfo=new ApiInfo(
                configProperties.getSwagger().getTitle(),
                configProperties.getSwagger().getDescription(),
                configProperties.getSwagger().getVersion(),
                configProperties.getSwagger().getTermsOfServiceUrl(),
                contact(),
                configProperties.getSwagger().getLicense(),
                configProperties.getSwagger().getLicenseUrl());
        return apiInfo;
    }

    public Contact contact(){
        Contact contact=new Contact(configProperties.getSwagger().getContactName(),configProperties.getSwagger().getContactEmail(),"");
        return contact;
    }

}
```

## application.yml配置swagger基本信息
```yml
swagger:
        title: SpringBoot Demo API
        description: SpringBoot Demo API documentation
        version: 0.0.1
        termsOfServiceUrl:
        contactName:
        contactUrl:
        contactEmail:
        license:
        licenseUrl:
```
## 控制器应用
```java
@RestController
@RequestMapping(value = "/api/v1/login")
@Api(tags = "外部操作api")
public class LoginResource {

    @Inject
    private TokenService tokenService;

    @RequestMapping(value = "login")
    @ApiOperation(value = "用户登录")
    public String login(String userName,String password){
        User user =new User();
        user.setId(1l);
        user.setUserName(userName);
        user.setPassword(password);
        user.setRealName("yuanwj");
        UserAuthentication authentication = tokenService.createToken(user);
        String token = authentication.getJwtToken();
        SecurityContextHolder.getContext().setAuthentication(authentication);
        return token;
    }
}

```

## 常用注解
```
@Api：修饰整个类，描述 Controller 的作用
@ApiOperation：描述一个类的一个方法，或者说一个接口
@ApiParam：单个参数描述
@ApiModel：用对象来接收参数
@ApiProperty：用对象接收参数时，描述对象的一个字段
```

## 访问
 http://ip:port/swagger-ui.html

## 效果
![swagger效果图](/images/swagger.png)