---
title: Mysql Explain详解
description:  Explain命令在解决数据库性能上是第一推荐使用命令，大部分的性能问题可以通过此命令来简单的解决，Explain可以用来查看SQL语句的执行效 果，可以帮助选择更好的索引和优化查询语句，写出更好的优化语句。
date: 2018-05-14 20:40:00
comments: true
tags: 
    - Mysql  
categories:
    - Mysql
---
# 描述及用法

MySQL 提供了一个 EXPLAIN 命令, 它可以对 SELECT 语句进行分析, 并输出 SELECT 执行的详细信息, 以供开发人员针对性优化.
EXPLAIN 命令用法十分简单, 在 SELECT 语句前加上 Explain 就可以了 

# 输出格式
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | Extra |
| - | -| -| -| -| -|-|-|-|-|-|
|1	|SIMPLE	|wms_order	| NULL |	index	| ix_order_status	|ix_uq_wmscode_storeid | 	168	|	40|	0.5|	Using where|

1. id  selete 查询标识符，每个查询自动分配
2. select_type 查询类型 结果集为
    - SIMPLE 简单查询
    - PRIMARY, 表示此查询是最外层的查询
    - UNION, 表示此查询是 UNION 的第二或随后的查询
    - DEPENDENT UNION, UNION 中的第二个或后面的查询语句, 取决于外面的查询
    - UNION RESULT, UNION 的结果
    - SUBQUERY, 子查询中的第一个 SELECT
3. table 查询所属表
4. partitions 查询分区
5. type 我理解为扫描类型
    - system: 表中只有一条数据. 这个类型是特殊的 const 类型.
    - const: 针对主键或唯一索引的等值查询扫描, 最多只返回一行数据. const 查询速度非常快, 因为它仅仅读取一次即可.
    - eq_ref: 此类型通常出现在多表的 join 查询, 表示对于前表的每一个结果, 都只能匹配到后表的一行结果. 并且查询的比较操作通常是 =, 查询效率较高. 
    - ref: 此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了 最左前缀 规则索引的查询.
    - range: 表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中.
    - index: 表示全索引扫描(full index scan), 和 ALL 类型类似, 只不过 ALL 类型是全表扫描, 而 index 类型则仅仅扫描所有的索引, 而不扫描数据.
    - ALL: 表示全表扫描,
    - **type 类型的性能关系如下:**
    ALL < index < range  < ref < eq_ref < const < system
    ALL 类型因为是全表扫描, 因此在相同的查询条件下, 它是速度最慢的.
    而 index 类型的查询虽然不是全表扫描, 但是它扫描了所有的索引, 因此比 ALL 类型的稍快.
    后面的几种类型都是利用了索引来查询数据, 因此可以过滤部分或大部分数据, 因此查询效率就比较高了
6. possible_keys 表示 MySQL 在查询时, 能够使用到的索引. 注意, 即使有些索引在 possible_keys 中出现, 但是并不表示此索引会真正地被 MySQL 使用到. MySQL 在查询时具体使用了哪些索引, 由 key 字段决定.
7. key 在当前查询时所真正使用到的索引.
8. key_len 查询时使用到索引的长度
9. ref 索引的哪一列被使用了
10. rows 估算 SQL 要查找到结果集需要扫描读取的数据行数.这个值非常直观显示 SQL 的效率好坏, 原则上 rows 越少越好.
11. Extra 额外的信息 比如Using filesort不能使用索引排序,导致产生filesort 