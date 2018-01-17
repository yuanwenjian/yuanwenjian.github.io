---
title: Spring Data Jpa多条件查询 and or组合查询
description:  类似 select * from user where name = '' and (email = '' or phone = "")
date: 2018-01-17 21:43:00
tags: 
    - Spring 
    - Spring Data
categories:
    - Spring
---

## 构建多条件查询
首先Repository 继承JpaSpecificationExecutor 构建查询代码如下

### 方法一
```java
Page<User> UserPage = userRepository.findAll(new Specification() {
    @Override
    public Predicate toPredicate(Root root, CriteriaQuery criteriaQuery, CriteriaBuilder criteriaBuilder) {
        List<Predicate> predicates = new ArrayList<Predicate>();
        if (StringUtils.isNotBlank(database)){
            predicates.add(criteriaBuilder.and(criteriaBuilder.equal(root.get("name"),database)));
        }
        if (StringUtils.isNotBlank(search)){
            Predicate pred=criteriaBuilder.and(criteriaBuilder.or(criteriaBuilder.like(root.get("email"),"%"+search+"%"),
                            criteriaBuilder.or(criteriaBuilder.like(root.get("phone"),"%"+search+"%"))));
            predicates.add(pred);
        }
        return criteriaBuilder.and(predicates.toArray(new Predicate[predicates.size()]));
        }
    },pageRequest);

```

### 方法二

```java
Page<User> UserPage = userRepository.findAll(new Specification() {
    @Override
    public Predicate toPredicate(Root root, CriteriaQuery criteriaQuery, CriteriaBuilder criteriaBuilder) {
        Predicate predicate = criteriaBuilder.conjunction();
        if (StringUtils.isNotBlank(database)){
            predicate = criteriaBuilder.and(criteriaBuilder.equal(root.get("name"),database));
        }
        if (StringUtils.isNotBlank(search)){
            predicate = criteriaBuilder.and(predicate,criteriaBuilder.or(criteriaBuilder.like(root.get("email"),search),
            criteriaBuilder.like(root.get("phone"),"%"+search+"%")));
        }
        return predicate;
        }
    },pageRequest);

```