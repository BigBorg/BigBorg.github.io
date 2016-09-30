---
layout: 	post
title:		"Borg的mysql笔记"
header-img:	"img/post-bg-2016.jpg"
date:		2016-09-29
author: 	"Borg"
catalog:	true
tags:
    mysql
---
# 数据库
```sql
create database [IF NOT EXISTS] <数据库名>
USE <数据库名>
ALTER DATABASE [数据库名] {[DEFAULT] CHARACTER SET <字符集名> | [DEFAULT] COLLATE <校对规则名>}
DROP DATABASE [IF EXISTS] <数据库名>
SHOW DATABASES [LIKE <pattern>]
```

# 表定义
```sql
CREATE TABLE <表名> ([表定义选项])[表选项][分区选项]
  eg. CREATE TABLE students(
        student_id INT NOT NULL AUTO_INCREMENT,
        student_name CHAR(50) NOT NULL,
        PRIMARY KEY(student_id)
        ) ENGINE = InnoDB
```

数据类型：BIT, TINYINT, BOOL/BOOLEAN, SMALLINT, MEDIUMINT, INT, INTEGER, BIGINT, FLOAT, DOUBLE, DECIMAL, DATE, DATETIME, TIMESTAMP, TIME, YEAR, CHAR, VARCHAR, TINYBLOB, TINYTEXT, BLOB, TEXT, ENUM  
引擎类型：SHOW ENGINE  

```sql
ALTER TABLE <表名> {ADD COLUMN 列名 类型| CHANGE COLUMN 旧列名 新列名 新类型| ALTER COLUMN 列名{SET DEFAULT 默认值| DROP DEFAULT}| MODIFY COLUMN 列名 类型| DROP COLUMN 列名| RENAME TO 新表名}
RENAME TABLE 表名 TO 新表名
CREATE TABLE 表名 [LIKE | AS 旧表名]  /*复制*/
DROP TABLE 表名[,表名2...]
SHOW TABLES [FROM 数据库名] [LIKE pattern]
SHOW [FULL] COLUMNS FROM tbl_name [FROM database_name][LIKE pattern] # DESCRIBE 快捷方式
```

# 表操作
```sql
SET NAMES GBK /* for Chinese characters */
INSERT INTO tbl_name [(column_name ...)] VALUES (value...)
REPLACE INTO tbl_name [(column_name ...)] {VALUES (value...)| SET col=val| SELECT ...}
DELETE FROM tbl_name [WHERE ...][ORDER BY col][LIMIT ...]
DELETE tbl1[,tbl2 FROM tbl1[,tbl2,tbl3...][WHERE condition]
TRUNCATE TABLE tbl_name
UPDATE tbl_name[,tbl2,tbl3...] SET col=val[,col2=val2...][WHERE ...][ORDER BY ...][LIMIT ...]
SELECT col1[,col2...] FROM tbl_name [WHERE ...][GROUP ...][HAVING ...][ORDER BY ...][LIMIT ...]
    CASE 
	WHEN condition THEN expression
	WHEN condition THEN expression
	...
	ELSE
      END [AS alias]
    CROSS JOIN
    tbl1 INNER JOIN tbl2 ON condition
    NATURAL JOIN
    LEFT OUTER JOIN / RIGHT OUTER JOIN
    WHERE
        WHERE ... {=| <| >| |<= |>= |<=> |<> |!= } {ALL |SOME |ANY} ...
        WHERE ... [NOT] LIKE ...	//通配符 % _
        WHERE ... [NOT] [REGEXP|RLIKE] ...    //LIKE 和 REGLIKE区别，LIKE 'A' 只匹配值为‘A’ 而 REGLIKE 'A' 匹配任何包含'A'的字符串
	WHERE ... [NOT] BETWEEN ... AND ...
        WHERE ... IS [NOT] NULL
        WHERE ... [NOT] IN (val1,val2...)
    GROUP BY ... HAVING ...  //HAVING 和 WHERE 区别, HAVING 分组后过滤，可包含聚合函数，WHERE相反
    UNION [DISTINCT |ALL] //默认DISTINCT
```

# 索引
```sql
INDEX / KEY
UNION
PRIMARY KEY
CREATE INDEX index_name ON tbl_name(col[(length)][ASC |DESC][USING BTREE|HASH]
SHOW INDEX FROM tbl_name [FROM db_name]
DROP INDEX index_name ON tbl_name
ALTER TABLE tbl_name {DROP PRIMARY KEY| DROP INDEX index_name |DROP FOREIGN KEY foreign_key}
```

在CREATE TABLE语句内使用：

```sql
KEY|INDEX [index_name][index_type](col1,col2...)
UNIQUE [INDEX|KEY] [index_name][index_type](col1,col2...)
FOREIGN KEY index_name col
```

在ALTER TABLE语句内使用：

```sql
ADD INDEX [index_name][index_type](col1,col2...)
ADD PRIMARY KEY [index_type](col1,col2...)
ADD UNIQUE [INDEX|KEY][index_name][index_type](col1,col2...)
ADD FOREIGN KEY [index_name](col1,col2...)
```

# 视图
```sql
CREATE VIEW view_name AS select_statement [WITH CHECK OPTION]	//修改视图时检查插入的数据是否符合WHERE设置的条件
DROP VIEW view1[,view2...]
ALTER VIEW view_name AS select_statement
SHOW CREATE VIEW view_name
```

# 完整性

## 实体完整性
```sql
PRIMARY KEY
UNIQUE
```

## 参照完整性
CREATE TABLE语句内：

```
FOREIGN KEY(key_name)
REFERENCES tbl2[(col[(length)])]
[ASC |DESC][MATCH FULL| MATCH PARTIAL| MATCH SIMPLE]
ON [DELETE |UPDATE] [RESTRICT |CASCADE |SET NULL |NO ACTION] //限制，级联，置空，不采取实施策略
```
外键指向的列/列组合必须是父表中的唯一确定的（取自主键或候选键）

## 用户定义完整性
```sql
NOT NULL
CHECK(expression)
```

## 命名完整性
```sql
CONSTRAINT 约束名 约束定义
```

## 更新完整性
```sql
ALTER TABLE语句内：
ADD FOREIGN KEY [索引名](col,...)
DROP PRIMARY KEY
DROP FOREIGN KEY 外键名
```

## 表维护
```sql
ANALYZE TABLE tbl1[,tbl2...]
CHECKSUM TABLE tbl1[,tbl2...] [QUICK |EXTENDED]
CHECK TABLE tbl1[,tbl2...][FOR UPGRADE| QUICK| FAST| MEDIUM| EXTENDED| CHANGED]
REPAIRE TABLE tbl1[,tbl2...][QUICK| EXTENDED| USE_FRM]
OPTIMIZE TABLE tbl1[,tbl2...]
```

# 触发器
```sql
CREATE TRIGGER trigger_name {BEFORE |AFTER} {INSERT |UPDATE |DELETE} ON tbl FOR EACH ROW trigger_body
DROP TRIGGER [IF EXISTS] [db_name.]trigger_name
INSERT触发器可用NEW虚拟表  
UPDATE触发器可用OLD, NEW虚拟表，OLD只读，涉及对触发表本身修改时只能用BEFORE  
DELETE触发器可用OLD虚拟表，只读  
同一表只能有6个触发器，不可修改只能删除重新创建
```

# 事件
```sql
SET GLOBAL EVENT_SCHEDULER = TRUE
CREATE EVENT event_name ON SCHEDULE schedule DO event_body
ALTER EVENT event_name [RENAME TO new_event_name][DO event_body][ENABLE | DISABLE]
DROP EVENT [IF EXISTS] event_name
```
schedule 格式 AT 子句或 EVERY 子句：

```sql
AT timestamp [+ INTERVAL time_span]...
EVERY time_span [STARTS timestamp [+ INTERVAL time_span]...] [ENDS timestamp [+ INTERVAL time_span]...]
```
time_span格式：  
YEAR, QUARTER, MONTH, DAY, HOUR, MINUTE, WEEK, SECOND, YEAR_MONTH, DAY_HOUR, DAY_MINUTE, DAY_SECOND, HOUR_MINUTE, HOUR_SECOND, MINUTE_SECOND  

event_body 多条语句应用BEGIN...END包裹

# 存储过程
```sql
DELIMITER ??
CREATE PROCEDURE procedure_name([parameters]) body
CALL precedure_name[([param1,param2...])]    //无参数时可省略括号
DROP {PROCEDURE|FUNCTION} [IF EXISTS] procedure_name
```
参数格式：
```sql
[IN |OUT |INOUT] para_name para_type
```
body内可用语句：

```sql
DECLARE var_name[,...] var_type [DEFAULT val]	//此处为局部变量，与用户变量、系统变量区别：用户变量用@开头(SET @var = val)，系统变量@@开头
SET var = val[,var2=val2...]	//声明后才可用SET
SELECT col[,...] INTO var[,...] ...

IF ... THEN ...
[ELSEIF ... THEN ...]
[ELSE ...]
END IF

CASE param
WHEN val THEN ...
[WHEN val2 THEN ...]
[ELSE ...]
END CASE

CASE
WHEN condition THEN ...
[WHEN condition THEN ...]
END CASE

<tag> LOOP
body		//LEAVE <tag> 退出循环，否则一直循环
END LOOP <tag>

<tag> WHILE condition DO
body
END WHILE<tag>

<tag> REPEAT
body
UNTIL condition
END REPEAT<tag>

DECLARE cursor_name CURSOR FOR select_statement
OPEN cursor_name
FETCH cursor_name INTO val[,val2...]	//取一行
CLOSE cursor_name
```

# 存储函数
```sql
DELIMITER ??							//修改语句结束标志
CREATE FUNCTION fun_name([var1 type1[,var2,type2,...]])
	RETURNS type
	body??							//body内语句用;隔开

SELECT fun_name([params])
DROP FUNCTION [IF EXISTS] fun_name
```

# 用户帐号管理
```sql
SELECT * FROM mysql.user
CREATE USER user_name [IDENTIFIED] BY [PASSWORD] key	//关键字PASSWORD取消对key的散列加密
DROP USER user1[,user2...]
RENAME USER user_old TO user_new
SET PASSWORD [FOR user] = { PASSWORD('password') | 'hash' }
GRANT privileges[(col,...)] ON {*|db}.{*|tbl} TO user [IDENTIFIED BY [PASSWORD] key] [WITH GRANT OPTION]
REVOKE privileges[(col,...)] ON {*|db}.{*|tbl} FROM user1[,user2...]
REVOKE ALL PRIVILEGES, GRANT OPTION FROM user1[,user2...]
```
