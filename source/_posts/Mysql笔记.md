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

SHOW VARIABLES LIKE '%max_allowed_packet%';

SET GLOBAL max_allowed_packet=268435456;