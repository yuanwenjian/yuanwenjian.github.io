---
title: Mysql 常用优化
description:  Mysql 性能优化按我理解尽量使用索引,因为索引实质为排好序的数据结构,完善的sql语句有时会大大加快查询速度
date: 2018-12-03 20:40:00
comments: true
tags: 
    - Mysql  
categories:
    - Mysql
---
# Mysql 常用优化策略

1. explian  善用explian 查询执行计划, 返回结果包括连接类型,查询条数,使用的索引.扫描的行数

2. sql 中in 条件不要太多, 有可能不走索引导致全表扫描

3. select 指定列,优点1.有可能走覆盖索引,2.避免不必要的消耗如(cpu,宽带)

4. ORDER BY中所有的列必须包含在相同的索引中并保持在索引中的排列顺序.ORDER BY中不能既有ASC也有DESC

5. 尽量避免or 字段,会走全表扫描

6. where 不要使用!= <> 不相等符号,有可能导致扫描全表

7. 最左前缀匹配原则 mysql 会从右匹配直到碰到范围(>,<,!=)等会停止匹配,=应该放在最左边

8. 使用like 时%like%,不会使用索引,like%能使用

9. where 条件禁止使用函数

10. where 禁止数据类型转换 比如id为1 ,where id='1' 禁止走索引

11. 使用联合索引时,包含第一列 比如联合索引(a,b,c) 应该保证条件有a

