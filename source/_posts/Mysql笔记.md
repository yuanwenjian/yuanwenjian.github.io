---
title: Mysql 常用命令记录
description:  记录mysql 常用命令，如导出，查看日志等
date: 2018-11-14 20:40:00
comments: true
tags: 
    - Mysql  
categories:
    - Mysql
---

## mysql导出文件



### The MySQL server is running with the –secure-file-priv option so it cannot execute this statement.

问题产生原因为导入文件路径与mysql限制路径不符

--secure-file-priv这个参数，这个参数的主要目的就是限制LOAD DATA INFILE或者SELECT INTO OUTFILE之类文件的目录位置，通过show variables like '%secure%'查看目录，

如果要解决这个问题，我们可以通过下面2种方式：

1. 将你要导入或导出的文件位置指定到你设置的路径里
2. 由于不能动态修改，我们可以修改my.cnf里关于这个选项的配置，然后重启即可。

### 导出语句
```sql
SELECT
	a.*, p.shop_name INTO OUTFILE '/var/lib/mysql-files/order.csv' FIELDS TERMINATED BY ',' LINES TERMINATED BY '\r\n'
FROM
	(
		SELECT
			w.*, p.order_expcode,
			p.package_weight
		FROM
			(
				SELECT
					order_code,
					order_id,
					order_omscode,
					shop_id,
					order_persion,
					order_mobile,
					order_address,
					print_date
				FROM
					wms_order
				WHERE
					print_date > '2018-11-11'
				AND print_date < '2018-11-14'
			) w
		LEFT JOIN wms_order_package p ON w.order_id = p.order_id
	) a
LEFT JOIN bas_shop p ON a.shop_id = p.shop_id;
```

## mysql 死锁问题
```sql
-- 查询是否锁表
show OPEN TABLES where In_use > 0;

-- 查看正在锁的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;

-- 查看等待锁的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;

-- 查看正在提交的事物
select * from information_schema.innodb_trx;
```

## mysql 开启日志
```sql
-- 查看日志目录
show variables like 'general_log%';
-- 慢日志
show variables like 'slow_query_log';

show variables like 'slow_query%';

-- 开启日志
set global general_log = on;

-- 关闭日志

set global general_log=off;
```

## 数据导出

```sql
SELECT
	p.order_expcode,
	o.order_code,
	s.sku_code,
	s.sku_name,
	d.order_buynum
INTO OUTFILE '/var/lib/mysql-files/order0521.csv' FIELDS TERMINATED BY ',' LINES TERMINATED BY '\r\n'
FROM
	`temp` t
JOIN wms_order_package p ON t.exp_code = p.order_expcode
JOIN wms_order o ON p.order_id = o.order_id
JOIN wms_order_detail d ON o.order_id = d.order_id
JOIN bas_sku s ON d.product_id = s.sku_id;
```

## mysql查询优化
1. 避免使用数据类型隐形转换查询
比如 where id ='11'

2. 禁止使用select * 
原因为
消耗更多的 CPU 和 IO 以网络带宽资源
无法使用覆盖索引
可减少表结构变更带来的影响

3. 避免使用子查询,尽量优化成join查询,
原因:
子查询的结果集无法使用索引，通常子查询的结果集会被存储到临时表中，不论是内存临时表还是磁盘临时表都不会存在索引，所以查询性能会受到一定的影响。特别是对于返回结果集比较大的子查询，其对查询性能的影响也就越大。

由于子查询会产生大量的临时表也没有索引，所以会消耗过多的 CPU 和 IO 资源，产生大量的慢查询。

4. 避免使用 JOIN 关联太多的表 
同一个 SQL 多关联（join）一个表，就会多分配一个关联缓存，如果在一个 SQL 中关联的表越多，所占用的内存也就越大。

如果程序中大量的使用了多表关联的操作，同时 join_buffer_size 设置的也不合理的情况下，就容易造成服务器内存溢出的情况，就会影响到服务器数据库性能的稳定性。

同时对于关联操作来说，会产生临时表操作，影响查询效率，MySQL 最多允许关联 61 个表，建议不超过 5 个。
5.  对应同一列进行 or 判断时，使用 in 代替 or
in 的值不要超过 500 个，in 操作可以更有效的利用索引，or 大多数情况下很少能利用到索引。

6. 禁止使用 order by rand() 进行随机排序

7. WHERE 从句中禁止使用函数
对列进行函数转换或计算时会导致无法使用索引




innodb与mysiam区别
1. 事务:innodb支持事务,MyISAM不支持
2. 行数: innodb不存储数据行数,myisam存储,select count(*) 很快
3. 索引不同:innodb是聚集索引,索引与数据存放在同一文件,
4. 外键支持: innodb支持外键,mysiam不支持
5. 锁不同: innodb支持行锁,mysiam只支持表锁
