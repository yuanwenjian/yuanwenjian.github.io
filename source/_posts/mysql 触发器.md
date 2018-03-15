---
title: Mysql触发器
description:  在指定表上，(insert(插入)、update(跟新)、delete(删除))事件动作,触发（After(之后)时机,Before(之前)），执行指定的一群或一个sql语句。
date: 2018-03-15 20:40:00
comments: true
tags: 
    - Trigger
    - Mysql  
categories:
    - Mysql
---
# 说明
在指定表上，(insert(插入)、update(跟新)、delete(删除))事件动作,触发（After(之后)时机,Before(之前)），执行指定的一群或一个sql语句。

# Trigger语法

```sql
DELIMITER $$
CREATE  Trigger trigger_name trigger_time trigger_event ON table_name FOR EACH ROW
begin 
trigger_body
end$$
```
**trigger_time** 触发器执行时间 可选值 AFTER,Before

**trigger_event** 激活触发器动作 可选值 update,insert,delete

**trigger_body** 触发器激活执行的动作 由Begin...End包裹

## New 与OLD关键字
在触发器主题中,可以使用New和Old访问受触发器影响的数据
1. 忽略大小写
2. 在insert触发器中只有New可用
3. 在update触发器中New,Old都可使用
4. 在delete触发器中只有Old可用
5. old.column只能查看,不能修改,但是new可以更改,这意味着在before的触发器中你可以使用set New.column=value,比如在before update使用,你可以修改要更新的值,这样的set语句在AFTER触发器不起作用,因为行已经发生改变

示例:
```sql
CREATE TRIGGER `emp_details_BINS` 
BEFORE INSERT 
ON emp_details FOR EACH ROW
-- Edit trigger body code below this line. Do not edit lines above this one
BEGIN
SET NEW.FIRST_NAME = TRIM(NEW.FIRST_NAME);
SET NEW.LAST_NAME = TRIM(NEW.LAST_NAME);
SET NEW.JOB_ID = UPPER(NEW.JOB_ID);END;
$$
//在更新时去掉空格
```

## 完整示例
每个科目分数全部或的,现在我们将更新表格的总分数，总分数和分数平均分,
1. PER_MARKS>90等级为EXCELLENT,
2. PER_MARKS>=75 AND PER_MARKS<90 为'VERY GOOD',
3. PER_MARKS>=60 AND PER_MARKS<75 -> 'GOOD'
4. PER_MARKS>=40 AND PER_MARKS<60 -> 'AVERAGE'
5. PER_MARKS<40-> 'NOT PROMOTED'

| STUDENT\_ID | NAME             | SUB1 | SUB2 | SUB3 | SUB4 | SUB5 | TOTAL | PER_MARKS | GRADE |
|-|-|-|-|-|-|-|-|-|-|
|          1 | Steven King      |    0 |    0 |    0 |    0 |    0 |     0 |      0.00 |       |

```sql
DELIMITER 
$$
CREATE TRIGGER `student_marks_BUPD` 
BEFORE UPDATE 
ON student_marks FOR EACH ROW
BEGIN 
SET NEW.TOTAL = NEW.SUB1 + NEW.SUB2 + NEW.SUB3 + NEW.SUB4 + NEW.SUB5; 
SET NEW.PER_MARKS = NEW.TOTAL/5;
IF NEW.PER_MARKS >=90 THEN
SET NEW.GRADE = 'EXCELLENT';
ELSEIF NEW.PER_MARKS>=75 AND NEW.PER_MARKS<90 THEN
SET NEW.GRADE = 'VERY GOOD';
ELSEIF NEW.PER_MARKS>=60 AND NEW.PER_MARKS<75 THEN
SET NEW.GRADE = 'GOOD';
ELSEIF NEW.PER_MARKS>=40 AND NEW.PER_MARKS<60 THEN
SET NEW.GRADE = 'AVERAGE';
ELSESET NEW.GRADE = 'NOT PROMOTED';
END IF;
END;
$$

```