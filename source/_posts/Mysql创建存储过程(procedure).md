---
title: Mysql存储过程
description: Procedure 是一个具有名称,参数列表,sql语句组成类似常规计算机语言的子程序,所有关系型数据库都支持存储过程,Mysql5 开始引入存储过程
date: 2018-03-14 20:40:00
comments: true
tags: 
    - Mysql
    - Procedure  
categories:
    - Mysql
---
# 存储过程简介
Procedure 是一个具有名称,参数列表,sql语句组成类似常规计算机语言的子程序,所有关系型数据库都支持存储过程,Mysql5 开始引入存储过程

# 创建存储过程
## 示例
```sql
DROP PROCEDURE IF EXISTS `test`; //如果存在删除
DELIMITER$$  //指定分隔符替代默认;分隔符
CREATE DEFINER='root' PROCEDURE test(IN param INTEGER(11)) 
BEGIN 
SELECT * FROM t_user WHERE email=param; 
END$$ //遇到$$执行语句
```
## 说明
1. DROP PROCEDURE 删除存储过程
2. DELIMITER 指定分隔符,默认为; 执行时遇到指定分隔符($$)才会执行
3. CREATE PROCEDURE 创建存储过程
4. DEFINER='root' 指定用户,默认为当前用户,可以省略
5. test 存储过程名称 

    1. 名称忽略大小写
    2. 同一数据库不能存在多个名称相同的存储过程
    3. 可以使用database.name 用来指定数据库
    4. 名称可以分隔,如果名称是分隔的，它可以包含空格。
    5. 名称最长为64字符
    6. 避免使用mysql内置函数
6. ()内为参数
    1. 格式为(IN|OUT|INOUT name type)
    2. IN 为传入参数,即使存储过程内修改,外部调用返回结果不会看到
    3. OUT 将存储过程内的值返回给调用者,
    4. INOUT 调用者初始化，可以由过程修改，并且在过程返回时调用程序可以看到过程所做的任何更改。
7. BEGIN...END块用来写复合语句

## OUT参数示例
```sql
DELIMITER$$ 
CREATE PROCEDURE my_proc_OUT (OUT highest_salary INT)
BEGIN
    SELECT MAX(MAX_SALARY) INTO highest_salary FROM JOBS;
END$$
调用
CALL my_proc_OUT(@M)
SELECT @M
```

## INOUT参数示例
```sql
CREATE PROCEDURE my_proc_INOUT (INOUT mfgender INT, IN emp_gender CHAR(1))
BEGIN
SELECT COUNT(gender) INTO mfgender FROM user_details WHERE gender = emp_gender;
END$$

调用
CALL my_proc_INOUT(@C,'M')
SELECT @C
```

# 流程控制
## 说明
MySQL可以使用IF，CASE，ITERATE，LEAVE，LOOP，WHILE和REPEAT流程控制,此外还支持RETURN
## IF流程控制
### 语法
>IF condition THEN statement(s)   
[ELSEIF condition THEN statement(s)] ...         
[ELSE statement(s)]  
END IF
### 示例
```sql
CREATE DEFINER=`root`@`127.0.0.1` 
PROCEDURE `GetUserName`(INOUT user_name varchar(16),
IN user_id varchar(16))
BEGIN
DECLARE uname varchar(16); //定义变量
SELECT name INTO uname
FROM user
WHERE userid = user_id;
IF user_id = "scott123" 
THEN
SET user_name = "Scott";
ELSEIF user_id = "ferp6734" 
THEN
SET user_name = "Palash";
ELSEIF user_id = "diana094" 
THEN
SET user_name = "Diana";
END IF;
END
```

## Case流程控制
### 语法
>CASE case_value    
WHEN when_value THEN statement_list         
[WHEN when_value THEN statement_list] ...         
[ELSE statement_list] END CASE
### 示例
```sql
DELIMITER $$
CREATE PROCEDURE `hr`.`my_proc_CASE` 
(INOUT no_employees INT, IN salary INT)
BEGIN
CASE
WHEN (salary>10000) 
THEN (SELECT COUNT(job_id) INTO no_employees 
FROM jobs 
WHERE min_salary>10000);
WHEN (salary<10000) 
THEN (SELECT COUNT(job_id) INTO no_employees 
FROM jobs 
WHERE min_salary<10000);
ELSE (SELECT COUNT(job_id) INTO no_employees 
FROM jobs WHERE min_salary=10000);
END CASE;
END$$
```
##  ITERATE Statement
### 作用
再次启动循环,ITERATE只能出现在LOOP，REPEAT和WHILE语句中
### 语法
ITERATE label
## LEAVE Statement
### 作用
LEAVE语句用于退出指定标签,如果LEAVE在语句最外面,则退出程序,LEAVE可用于BEGIN ... END或循环结构（LOOP，REPEAT，WHILE）
### 语法
LEAVE label

## LOOP Statement

### 语法
>[begin_label:] 
LOOP       
statement_list  
END LOOP 
[end_label]
### 作用
LOOP用于创建语句列表的重复执行,statement_list是一个或多个使用;结尾的语句,重复执行列表的语句,直至循环结束,通常，LEAVE语句用于退出循环结构。也可以使用RETURN，但它完全退出函数。 LOOP语句可以被标记
### 示例
```sql
DELIMITER $$
CREATE PROCEDURE `my_proc_LOOP` (IN num INT)
BEGIN
DECLARE x INT;
SET x = 0;
loop_label: LOOP
INSERT INTO number VALUES (rand());
SET x = x + 1;
IF x >= num 
THEN
LEAVE loop_label;
END IF;
END LOOP;
END$$
```

## WHILE Statement
### 语法
>[begin_label:] WHILE search_condition DO
    statement_list
END WHILE [end_label]
### 示例
```sql
DELIMITER $$
CREATE PROCEDURE my_proc_WHILE(IN n INT)
BEGIN
SET @sum = 0;
SET @x = 1;
WHILE @x<n 
DO
   IF mod(@x, 2) <> 0 THEN   
SET @sum = @sum + @x;   
END IF;   
SET @x = @x + 1;   
END WHILE;
END$$
```

# 修改存储过程 ALTER PROCEDURE
可以更改存储过程的特征,但是无法更改存储的实体及参数,如果想修改,可以使用drop再重新创建

# 存储过程权限
存储过程及视图允许使用DEFINER属性指定所属用户
只有拥有SUPER权限时，才能指定除自己帐户以外的DEFINER值

# 参考文献
[MySQL Procedure][资源]


[资源]: https://www.w3resource.com/mysql/mysql-procedure.php