---
title: SpringDataJpa返回List<Map<String,Object>>对象
date: 2018-01-18 13:32:26
tags:
    - Spring
    - Spring Data
description: Spring data Jpa返回Map对象
categories:
    - Spring 
    - Web
---

## 前言
Spring Data Jpa通常会只取几个字段值而不是全部字段,类似于
```sql
 select u.name ,u.email from t_user ;
```
在Jpa 中如何实现


```java
//原生sql返回结果为Object[]数组
@Query("select u.name ,u.email from t_user",nativeQuery=true)
List<Map<String,Object>> findUser();

//正确用法 
@Query("select new map (u.name as name,u.email as email) from User")
List<Map<String,Object>> findUser();
```

