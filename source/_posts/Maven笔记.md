---
title: Maven笔记
description:  maven问题记录汇总
date: 2018-04-20 13:25:00
comments: true
tags: 
    - Maven
    - tool
categories:
    - Tools
---

# Maven记录汇总

## maven 打包跳过测试编译
```bash
mvn clean install -Dmaven.test.skip=true
```

## maven scope详解
### 可选值
|complie| provided|runtime|test|
|-|-|-|-|

### 作用
- compile 
默认值，如果没有指定，则使用该范围。编译依赖关系在项目的所有类路径中都可用。而且，这些依赖关系会传播到依赖项目
- provided
类似于compile 但是打包时有JD

### 传递关系
![scope][scopeType]

[scopeType]: /images/scope.jpg

