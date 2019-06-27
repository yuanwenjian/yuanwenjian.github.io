---
title: Mybatis Generator应用及配置详解
description:  Mybatis Generator应用及配置详解
date: 2019-05-14 20:40:00
comments: true
tags: 
    - Java  
categories:
    - Mybatis
---
1. resource目录下新建文件generatorConfig.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 加载配置项货配置文件 -->
    <properties resource="application.properties"/>
    <!--<classPathEntry-->
            <!--location="/Users/wangfei/.m2/repository/mysql/mysql-connector-java/5.1.15/mysql-connector-java-5.1.15.jar"/> -->
            <!-- targetRuntime 指定为MyBatis3 会生成Simple类，通常不需要，可以设置为mybatis3simple -->
    <context id="mysqlTb" targetRuntime="MyBatis3" defaultModelType="flat">

        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>


        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="${datasource.url}"
                        userId="${datasource.username}"
                        password="${datasource.password}"/>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!--生成数据表对应的model-->
        <javaModelGenerator targetProject="src/main/java"
                            targetPackage="com.ct.miniprogram.model"> <!--目标项目，指定一个存在的目录下，生成的内容会放到指定目录中，如果目录不存在，MBG不会自动建目录-->
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!--生成xml文件-->
        <sqlMapGenerator targetPackage="mybatis/mapper"
                         targetProject="src/main/resources"/>

        <!--生成mapper-->
        <javaClientGenerator type="XMLMAPPER" targetProject="src/main/java" targetPackage="com.ct.miniprogram.mapper">
            <property name="enableSubPackages" value="true"/>
            <property name="exampleMethodVisibility" value="default"/>
        </javaClientGenerator>


        <table tableName="bas_country" domainObjectName="CountryEntity">
            <generatedKey column="cty_id" sqlStatement="Mysql" identity="true"/>
        </table>

    </context>
</generatorConfiguration>
```

2. pom文件新增插件

```xml
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.5</version>
                <configuration>
                    <!-- 指定上述配置文件路径 -->
                    <configurationFile>src/main/resources/generatorConfig/generatorConfig.xml</configurationFile>
                </configuration>
                <!-- 指定jdbc驱动,可以generatorConfig.xml中设置 -->
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>5.1.18</version>
                    </dependency>
                </dependencies>
            </plugin>
```