# SQL

## 基本语法

1. 最基本的 SQL 命令：
    * SELECT - 从数据库中提取数据
        ```sql
            SELECT column_name1,column_name2 FROM table_name;
            SELECT * FROM table_name;
        ```
    * UPDATE - 更新数据库中的数据
        ```sql
            UPDATE table_name SET column1=value1,column2=value2,... WHERE some_column=some_value;
        ```
    * DELETE - 从数据库中删除数据
        ```sql
            DELETE FROM table_name WHERE some_column=some_value;
        ```
    * INSERT INTO - 向数据库中插入新数据
        ```sql
            INSERT INTO table_name VALUES (value1,value2,value3,...);
            INSERT INTO table_name (column1,column2,column3,...) VALUES (value1,value2,value3,...);
        ```
    * CREATE DATABASE - 创建新数据库
        ```sql
            create database test;
        ```
    * ALTER DATABASE - 修改数据库
    * CREATE TABLE - 创建新表
        ```sql
            CREATE TABLE Persons
            (
                PersonID int,
                LastName varchar(255),
                FirstName varchar(255),
                Address varchar(255),
                City varchar(255)
            );
        ```
    * ALTER TABLE - 变更（改变）数据库表
        ```sql
            ALTER TABLE table_name ADD column_name datatype;
            ALTER TABLE table_name DROP COLUMN column_name
        ```
    * DROP TABLE - 删除表
        ```sql
            DROP TABLE table_name;
            DROP DATABASE database_name;
        ```
    * CREATE INDEX - 创建索引（搜索键）

        在表中创建索引，以便更加快速高效地查询数据。

        用户无法看到索引，它们只能被用来加速搜索/查询。

        更新一个包含索引的表需要比更新一个没有索引的表花费更多的时间，这是由于索引本身也需要更新。因此，理想的做法是仅仅在常常被搜索的列（以及表）上面创建索引。

        ```sql
            CREATE INDEX index_name ON table_name (column_name);
            CREATE UNIQUE INDEX index_name ON table_name (column_name)
        ```

        [详细了解索引](https://www.cnblogs.com/hyd1213126/p/5828937.html)

    * DROP INDEX - 删除索引

2. SQL更多用法：

    * like与通配符

        ```sql
            SELECT * FROM Websites WHERE name LIKE 'G%';
            --选取 name 以 "G"、"F" 或 "s" 开始
            SELECT * FROM Websites WHERE name REGEXP '^[GFs]';
            --选取 name 以 A 到 H 字母开头
            SELECT * FROM Websites WHERE name REGEXP '^[A-H]';
            --选取 name 不以 A 到 H 字母开头
            SELECT * FROM Websites WHERE name REGEXP '^[^A-H]';
            --将搜索下列字符串：Carsen、Karsen、Carson 和 Karson（如 Carson）。
            SELECT * FROM Websites WHERE name LIKE '[CK]ars[eo]n';
            --将搜索以字符串 inger 结尾、以从 M 到 Z 的任何单个字母开头的所有名称（如 Ringer）。
            SELECT * FROM Websites WHERE name LIKE '[M-Z]inger';
            --将搜索以字母 M 开头，并且第二个字母不是 c 的所有名称（如MacFeather）。
            SELECT * FROM Websites WHERE name LIKE 'M[^c]%';
            --'%a'          以a结尾的数据
            --'a%'          以a开头的数据
            --'%a%'         含有a的数据
            --'_a_'         三位且中间字母是a的
            --'_a'          两位且结尾字母是a的
            --'a_'          两位且开头字母是a的
            --[charlist]	字符列中的任何单一字符
            --[^charlist]   不在字符列中的任何单一字符
            --[!charlist]	不在字符列中的任何单一字符
        ```

    * in

        ```sql
            select * from Websites where name in ('value1','value2');
            select * from Websites where name='value1' or name='value2';
            --mysql会转化为exists，弊端：a表(外表)使用不了索引，必须全表扫描，因为是拿a表的数据到b表查。而且必须得使用a表的数据到b表中查（外表到里表中），顺序是固定死的。
            select * from a where exists(select * from b where b.id=a.id );
        ```

        in的效率提升
        ```sql
            select * from a where id in (select id from b );
            改为
            select * from a inner join b on a.id=b.id; 
        ```

        如果in条件规模小，则考虑使用in，否则考虑使用inner join

    * between

        mysql和mssql 字母和数字包含边界值,日期不包含右边界，因为‘2016-05-14’为‘2016-05-14 00:00:00’，所以不包含结束当天。

        ```sql
            SELECT column_name(s) FROM table_name WHERE column_name BETWEEN value1 AND value2;
            SELECT * FROM Websites WHERE alexa NOT BETWEEN 1 AND 20;
            SELECT * FROM Websites WHERE name BETWEEN 'A' AND 'H';
            SELECT * FROM Websites WHERE name NOT BETWEEN 'A' AND 'H';
            SELECT * FROM access_log WHERE date BETWEEN '2016-05-10' AND '2016-05-14';
        ```

    * as

        在下面的情况下，使用别名很有用：
        * 在查询中涉及超过一个表
        * 在查询中使用了函数
        * 列名称很长或者可读性差
        * 需要把两个列或者多个列结合在一起

        ```sql
            SELECT column_name AS column_alias_name FROM table_name AS table_alias_name;
        ```

    * join

        * INNER JOIN：如果表中有至少一个匹配，则返回行
        * LEFT JOIN：即使右表中没有匹配，也从左表返回所有的行
        * RIGHT JOIN：即使左表中没有匹配，也从右表返回所有的行
        * FULL OUTER JOIN：只要其中一个表中存在匹配，则返回行(MySQL中不支持 FULL OUTER JOIN,使用UNION实现)

        ```sql
            --on条件成立，且table1与table2不能出现null，组合成新表
            SELECT column_name(s) FROM table1 INNER JOIN table2 ON table1.column_name=table2.column_name;
            --on条件成立，且table1不能出现null，table2可出现null，组合成新表
            SELECT column_name(s) FROM table1 LEFT JOIN table2 ON table1.column_name=table2.column_name;
            --on条件成立，且table1可出现null，table2不能出现null，组合成新表
            SELECT column_name(s) FROM table1 RIGHT JOIN table2 ON table1.column_name=table2.column_name;
            --on条件成立，且table1与table2可出现null，组合成新表
            SELECT column_name(s) FROM table1 FULL OUTER JOIN table2 ON table1.column_name=table2.column_name;
            --mysql实现FULL OUTER JOIN
            SELECT column_name(s) FROM table1 LEFT JOIN table2 ON table1.column_name=table2.column_name
            UNION
            SELECT column_name(s) FROM table1 RIGHT JOIN table2 ON table1.column_name=table2.column_name
        ```

    * union

    * select into

    * insert into select

    * 约束

    * drop

    * 视图

    * is

    * 数据类型

3. SQL函数


## 关系型数据库

MySQL使用MySQL Workbench,MsSQL使用MicroSoft SQL Server Management Studio，进行图形化操作数据库。

数据库大小写不敏感。

### MySQL

1. 创建数据库并使用
```sql
CREATE DATABASE [ IF NOT EXISTS] test [ CHARACTER SET utf8 ];
use test;
```

2. 创建数据表
```sql
create table tutorials_tbl(
   tutorial_id INT NOT NULL AUTO_INCREMENT,
   tutorial_title VARCHAR(100) NOT NULL,
   tutorial_author VARCHAR(40) NOT NULL,
   submission_date DATE,
   PRIMARY KEY ( tutorial_id )
);
```

3. 修改数据表
```sql
ALTER TABLE table_name
MODIFY COLUMN column_name datatype

ALTER TABLE testalter_tbl  DROP i;

ALTER TABLE testalter_tbl ADD i INT;

-- 指定新增字段的位置 
ALTER TABLE testalter_tbl DROP i;
ALTER TABLE testalter_tbl ADD i INT FIRST; -- 设定位第一列
ALTER TABLE testalter_tbl DROP i;
ALTER TABLE testalter_tbl ADD i INT AFTER c; -- 设定位于某个字段之后

--  在 CHANGE 关键字之后，紧跟着的是你要修改的字段名，然后指定新字段名及类型。
ALTER TABLE testalter_tbl CHANGE i j BIGINT;

ALTER TABLE testalter_tbl MODIFY j BIGINT NOT NULL DEFAULT 100;
```

4. 修改数据库
```sql
--1.将名为"例二数据库"的数据库改名为"例七数据库"
alter database 例二数据库
modify name = 例七数据库

--2.为"例六数据库"增加一个数据文件
alter database 例六数据库
add file (
    name=增加的数据文件,
    filename='c:\dbtest\例六数据库增加的数据文件.ndf'
)
--3.为"例六数据库"增加一个日志文件

alter database 例六数据库
add log file (
    name=例六增加的日志文件,
    filename='c:\dbtest\例六增加的日志文件.ldf',
    size=3MB,
    maxsize=50MB,
    filegrowth=10%
)

--4.将"例六数据库"名为"增加的数据文件"的数据库文件改名
alter database 例六数据库
modify file (
    name=增加的数据文件,
    newname=例六数据文件,
    filename='c:\dbtest\例六数据文件.ndf'
)
--5.修改"例六数据库"的排序规则
alter database 例六数据库
collate Chinese_PRC_CI_AS_KS

--6.在"例六数据库"里删除一个数据文件
alter database 例六数据库
remove file 例六数据文件

--7.在"例六数据库"里添加一个文件组
alter database 例六数据库
add filegroup 例十三文件组

--8.在"例六数据库"里为一个文件组改名
alter database 例六数据库
modify filegroup 例十三文件组
name=例十四文件组

--9.在"例六数据库"里添加一个数据文件到一个文件组，并将该文件祖设为默认文件组。
--alter database一次只能修改数据库的一个属性
alter database 例六数据库
add file 
(
    name=例十五数据文件,
    filename='c:\dbtest\例十五数据文件.ndf'
)
to filegroup 例十四文件组
go
alter database 例六数据库
modify filegroup 例十四文件组 default
go

--10.在"例六数据库"里删除"例十四文件组"。
alter database 例六数据库
modify filegroup [primary] default
--将primary文件组设为默认文件组
go
alter database 例六数据库
remove file 例十五数据文件
--删除"例十四文件组"中包含的"例十五数据文件"
go
alter database 例六数据库
remove filegroup 例十四文件组
--删除"例十四文件组"
go

--11.将"例六数据库"里一个文件组设为只读的。
显示代码打印
alter database 例六数据库
add filegroup 例十七文件组
--先添加一个文件组，因为primary文件组不能设为只读
go
alter database 例六数据库
add file 
(
    name=例十七数据文件,
    filename='c:\dbtest\例十七数据文件.ndf'
)
to filegroup 例十七文件组
--添加一个文件到文件组中，因为空文件组不能设为只读
go
alter database 例六数据库
modify filegroup 例十七文件组 read_only
--将文件组设为只读
go

--12.将"例六数据库"设为只有一个用户可访问
alter database 例六数据库
set single_user

--13.设置"例六数据库"可自动收缩
alter database 例六数据库
set auto_shrink on
```

5. 删除表索引
```sql
ALTER TABLE table_name DROP INDEX index_name
```

6. 查询10-20条记录
```sql
select top10 * from (select top 20 * from order by column) order by column desc
```

[详细了解mysql索引](https://www.cnblogs.com/whgk/p/6179612.html)

### MsSQL

1. 创建数据库
```sql
use master
go

if exists(select * from sysdatabases where name='student')
drop database student
go

-- 数据库文件
create database student
on
(
    name='student_data',
    filename='student_data.mdf',
    size=10mb,
    maxsize=100mb,
    filegrowth=1mb
)

-- 日志文件
log on
(
    name='student_log',
    filename='student_log.ldf',
    size=10mb,
    maxsize=100mb,
    filegrowth=1mb
)
```

2. 创建数据表
```sql
USE School

IF EXISTS (SELECT *FROM sysobjects WHERE name='teacher')
	DROP TABLE teacher

CREATE TABLE Teacher
(
    Id INT IDENTITY(1,1),
    NAME NVARCHAR(50) NOT NULL,
    ClassId INT NOT NULL,
    Gender BIT NOT NULL,
    Age INT,
    Salary MONEY,
    Birthary DATETIME
)

```

3. 修改数据表
```sql
ALTER TABLE table_name
ALTER COLUMN column_name datatype
```

4. 修改数据库
```sql
alter datebase db_name
add file
(
    name = 'file_name',
    filename = 'F:\data\file_name.ndf',
    size = 2MB,
    maxsize = 100MB,
    filegrowth = 5MB
) to filegroup **

alert database db_name
modify file
(
    name = file_name,--file_name 是要修改的数据库文件名
    size = 4MB
)
```

5. 删除表索引
```sql
DROP INDEX table_name.index_name
```

6. 查询10-20条记录
```sql
select * from tablename where LIMIT 9,10
```

[详细了解mssql索引](https://www.cnblogs.com/Brambling/p/6754993.html)

## 非关系型数据库

### NoSQL

