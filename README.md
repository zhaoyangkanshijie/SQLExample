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

        约束：
        * NOT NULL - 指示某列不能存储 NULL 值。
        * UNIQUE - 保证某列的每行必须有唯一的值。
        * PRIMARY KEY - NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。
        * FOREIGN KEY - 保证一个表中的数据匹配另一个表中的值的参照完整性。
        * CHECK - 保证列中的值符合指定的条件。
        * DEFAULT - 规定没有给列赋值时的默认值。

        删除约束：
        ```sql
            alter table x modify column_name null;
            alter table x modify column_name not null;

            ALTER TABLE Persons ADD CONSTRAINT uc_PersonID UNIQUE (P_Id,LastName);
            --撤销 UNIQUE:
            --MySQL
            ALTER TABLE Persons DROP INDEX uc_PersonID
            --MsSQL
            ALTER TABLE Persons DROP CONSTRAINT uc_PersonID

            ALTER TABLE Persons ADD CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName);
            --撤销 PRIMARY KEY:
            --MySQL
            ALTER TABLE Persons DROP PRIMARY KEY
            --MsSQL
            ALTER TABLE Persons DROP CONSTRAINT pk_PersonID

            ALTER TABLE Orders ADD FOREIGN KEY (P_Id) REFERENCES Persons(P_Id);
            --命名 FOREIGN KEY 约束，并定义多个列的 FOREIGN KEY 约束
            ALTER TABLE Orders ADD CONSTRAINT fk_PerOrders FOREIGN KEY (P_Id) REFERENCES Persons(P_Id)
            --撤销 FOREIGN KEY:
            --MySQL
            ALTER TABLE Orders DROP FOREIGN KEY fk_PerOrders
            --MsSQL
            ALTER TABLE Orders DROP CONSTRAINT fk_PerOrders

            ALTER TABLE Persons ADD CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes');
            --撤销 CHECK:
            --MySQL
            ALTER TABLE Persons DROP CHECK chk_Person
            --MsSQL
            ALTER TABLE Persons DROP CONSTRAINT chk_Person

            --添加default
            --MySQL
            ALTER TABLE Persons ALTER City SET DEFAULT 'SANDNES'
            --MsSQL
            ALTER TABLE Persons ADD CONSTRAINT ab_c DEFAULT 'SANDNES' for City
            --撤销default
            --MySQL
            ALTER TABLE Persons ALTER City DROP DEFAULT
            --MsSQL
            ALTER TABLE Persons ALTER COLUMN City DROP DEFAULT
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

        UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。每个 SELECT 语句中的列的顺序必须相同。UNION 默认选取不同的值。UNION ALL允许重复的值。

        ```sql
            SELECT column_name(s) FROM table1
            UNION
            SELECT column_name(s) FROM table2;

            SELECT column_name(s) FROM table1
            UNION ALL
            SELECT column_name(s) FROM table2;
        ```

    * select into

        从一个表复制数据，然后把数据插入到另一个新表中

        ```sql
            SELECT column_name(s) INTO newtable FROM table1;
        ```

        mysql替代方法
        ```sql
            CREATE TABLE 新表
            AS
            SELECT * FROM 旧表 
        ```

    * insert into select

        从一个表复制数据，插入到一个已存在的表中,目标表中任何已存在的行都不会受影响。

        ```sql
            INSERT INTO table1 (a, b) SELECT c, d FROM table2 WHERE id=1;
        ```

    * drop

        ```sql
            drop 类型(表，列，索引) 名字
        ```

    * alert

        ```sql
            alert 类型(表，列，索引) 名字 操作(add,drop,modify) 类型(表，列，索引) 名字 [新值]
        ```

    * 视图

        视图是基于 SQL 语句的结果集的可视化的表。
        视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。

        ```sql
            CREATE VIEW view_name AS SELECT column_name(s) FROM table_name WHERE condition;
            CREATE VIEW [Products Above Average Price] AS SELECT ProductName,UnitPrice FROM Products WHERE UnitPrice>(SELECT AVG(UnitPrice) FROM Products)

            CREATE OR REPLACE VIEW view_name AS SELECT column_name(s) FROM table_name WHERE condition
            DROP VIEW view_name
        ```

    * is

        ```sql
            SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NULL;
            SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NOT NULL
        ```

    * 数据类型

        |   数据类型   |          解析             |   mssql   |   mysql   |
        |-------------|--------------------------|-----------|-----------|
        |CHARACTER(n) |字符/字符串。固定长度 n。     |           |           |
        |VARCHAR(n) 或CHARACTER VARYING(n)|字符/字符串。可变长度。最大长度 n。|||
        |BINARY(n)    |二进制串。固定长度 n。       |           |            |
        |BOOLEAN	  |存储 TRUE 或 FALSE 值      |    bit    | tinyint(1) |
        |VARBINARY(n) 或 BINARY VARYING(n)|二进制串。可变长度。最大长度 n。|||
        |INTEGER(p)	  |整数值（没有小数点）。精度 p。 |           |            |
        |SMALLINT	  |整数值（没有小数点）。精度 5。 |           |            |
        |INTEGER	  |整数值（没有小数点）。精度 10。|    int    |int/integer |
        |BIGINT	      |整数值（没有小数点）。精度 19。|           |            |
        |DECIMAL(p,s) |精确数值，精度 p，小数点后位数 s。例如：decimal(5,2) 是一个小数点前有 3 位数小数点后有 2 位数的数字。         |           |            |
        |NUMERIC(p,s) |精确数值，精度 p，小数点后位数 s。（与 DECIMAL 相同）|||
        |FLOAT(p)     |近似数值，尾数精度 p。一个采用以 10 为基数的指数计数法的浮点数。该类型的 size 参数由一个指定最小精度的单一数字组成。|           |           |
        |REAL         |近似数值，尾数精度 7。        |           |            |
        |FLOAT        |近似数值，尾数精度 16。       |Float/Real |   Float    |
        |DOUBLE PRECISION|近似数值，尾数精度 16。    |           |            |
        |DATE         |存储年、月、日的值。          |           |            |
        |TIME         |存储小时、分、秒的值。         |           |            |
        |TIMESTAMP    |存储年、月、日、小时、分、秒的值。|           |           |
        |INTERVAL     |由一些整数字段组成，代表一段时间，取决于区间的类型。|||
        |ARRAY        |元素的固定长度的有序集合       |           |            |
        |MULTISET     |元素的可变长度的无序集合       |           |            |
        |XML          |存储 XML 数据               |           |            |
        |Currency     |货币                        |    Money  |numeric(15,4)|
        |string (fixed)|固定长度字符串              |    Char   |    Char    |
        |string (variable)|可变长度字符串           |  Varchar  |  Varchar   |
        |binary object|二进制对象                  |Binary (fixed up to 8K) /Varbinary (<8K)/Image (<2GB)| Blob/Text |

3. SQL函数

    SQL Aggregate 函数计算从列中取得的值，返回一个单一的值。
    * AVG() - 返回平均值
    
        ```sql
            SELECT site_id, count FROM access_log WHERE count > (SELECT AVG(count) FROM access_log);
        ```

    * COUNT() - 返回行数

        ```sql
            SELECT COUNT(column_name) FROM table_name;--排除null
            SELECT COUNT(*) FROM table_name;
            SELECT COUNT(DISTINCT column_name) FROM table_name;--不同值
        ```

    * MAX() - 返回最大值

        ```sql
            SELECT MAX(column_name) FROM table_name;
        ```

    * MIN() - 返回最小值

        ```sql
            SELECT MIN(column_name) FROM table_name;
        ```
        
    * SUM() - 返回总和

        ```sql
            SELECT SUM(column_name) FROM table_name;
        ```
        

    SQL Scalar 函数基于输入值，返回一个单一的值。
    * UCASE() - 将某个字段转换为大写

        ```sql
            SELECT UCASE(column_name) FROM table_name;
            --mssql
            SELECT UPPER(column_name) FROM table_name;
        ```
        
    * LCASE() - 将某个字段转换为小写

        ```sql
            SELECT LCASE(column_name) FROM table_name;
            --mssql
            SELECT LOWER(column_name) FROM table_name;
        ```
        
    * MID() - 从某个文本字段提取从start开始的length个字符，MySql 中使用

        ```sql
            SELECT MID(column_name,start[,length]) FROM table_name;
        ```
        
    * SubString(字段，1，end) - 从某个文本字段提取字符
    * LEN() - 返回某个文本字段的长度

        ```sql
            SELECT LEN(column_name) FROM table_name;
            --mysql
            SELECT LENGTH(column_name) FROM table_name;
        ```
        
    * ROUND() - 对某个数值字段进行指定小数位数的四舍五入

        ```sql
            SELECT ROUND(column_name,decimals) FROM table_name;
        ```
        
    * NOW() - 返回当前的系统日期和时间

        ```sql
            SELECT name, url, Now() AS date FROM Websites;
        ```
        
    * FORMAT() - 格式化某个字段的显示方式

        ```sql
            SELECT name, url, DATE_FORMAT(Now(),'%Y-%m-%d') AS date FROM Websites;
        ```
        

    聚合
    * GROUP BY

        ```sql
            SELECT site_id, SUM(access_log.count) AS nums FROM access_log GROUP BY site_id;
        ```
        
    * HAVING

        ```sql
            SELECT Websites.name, Websites.url, SUM(access_log.count) AS nums FROM (access_log INNER JOIN Websites ON access_log.site_id=Websites.id) GROUP BY Websites.name HAVING SUM(access_log.count) > 200;
        ```
        

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

7. 处理NULL值为0
```sql
SELECT ProductName,UnitPrice*(UnitsInStock+ISNULL(UnitsOnOrder,0)) FROM Products
```

8. 数据类型

|    数据类型    |                      解析                      |     储存    |
|--------------|------------------------------------------------|------------|
|char(n)	   |固定长度的字符串。最多 8,000 个字符。                |     n      |
|varchar(n)    |可变长度的字符串。最多 8,000 个字符。                |2 bytes + n |
|varchar(max)  |可变长度的字符串。最多 1,073,741,824 个字符。        |2 bytes + n |
|text          |可变长度的字符串。最多 2GB 文本数据。                |4 bytes + n |
|nchar         |固定长度的 Unicode 字符串。最多 4,000 个字符。       |     2n     |
|nvarchar      |可变长度的 Unicode 字符串。最多 4,000 个字符。       |            |
|nvarchar(max) |可变长度的 Unicode 字符串。最多 536,870,912 个字符。 |            |
|ntext         |可变长度的 Unicode 字符串。最多 2GB 文本数据。	     |            |
|bit           |允许 0、1 或 NULL                                |            |
|binary(n)     |固定长度的二进制字符串。最多 8,000 字节。             |            |
|varbinary     |可变长度的二进制字符串。最多 8,000 字节。             |            |
|varbinary(max)|可变长度的二进制字符串。最多 2GB。                   |            |
|image         |可变长度的二进制字符串。最多 2GB。                   |            |
|tinyint       |允许从 0 到 255 的所有数字。                       |   1 byte   |
|smallint      |允许介于 -2^15 与 2^15-1 的所有数字。               |  2 bytes   |
|int           |允许介于 -2^31 与 2^31-1 的所有数字。               |  4 bytes   |
|bigint        |允许介于 -2^63 与 2^63-1 之间的所有数字。            |  8 bytes   |
|decimal(p,s)  |固定精度和比例的数字。<br>允许从 -10^38 +1 到 10^38 -1 之间的数字。<br>p 参数指示可以存储的最大位数（小数点左侧和右侧）。<br>p 必须是 1 到 38 之间的值。默认是 18。<br>s 参数指示小数点右侧存储的最大位数。<br>s 必须是 0 到 p 之间的值。默认是 0。|5-17 bytes|
|numeric(p,s)  |同上                                             | 5-17 bytes |
|smallmoney    |介于 -2^31 与 2^31-1 之间的货币数据。               |   4 bytes  |
|money         |介于 --2^63 与 -2^63-1 之间的货币数据。             |   8 bytes  |
|float(n)      |从 -1.79E + 308 到 1.79E + 308 的浮动精度数字数据。<br>n 参数指示该字段保存 4 字节还是 8 字节。float(24) 保存 4 字节，而 float(53) 保存 8 字节。n 的默认值是 53。|4 或 8 bytes|
|real          |从 -3.40E + 38 到 3.40E + 38 的浮动精度数字数据。   |   4 bytes  |
|datetime      |从1753年1月1日到9999年12月31日，精度为 3.33 毫秒。   |   8 bytes  |
|datetime2     |从1753年1月1日到9999年12月31日，精度为 100 纳秒。    | 6-8 bytes  |
|smalldatetime |从1900年1月1日到2079年6月6日，精度为 1 分钟。	     |   4 bytes  |
|date          |仅存储日期。从0001年1月1日 9999年12月31日。          |   3 bytes  |
|time          |仅存储时间。精度为 100 纳秒。                       | 3-5 bytes  |
|datetimeoffset|与 datetime2 相同，外加时区偏移。                   | 8-10 bytes |
|timestamp     |存储唯一的数字，每当创建或修改某行时，该数字会更新。timestamp 值基于内部时钟，不对应真实时间。每个表只能有一个 timestamp 变量。||
|sql_variant   |存储最多8,000字节不同数据类型的数据，除了text、ntext以及timestamp。||
|uniqueidentifier|存储全局唯一标识符 (GUID)。                       |           |
|xml           |存储 XML 格式化数据。最多 2GB。                     |           |
|cursor        |存储对用于数据库操作的指针的引用。                    |           |
|table         |存储结果集，供稍后处理。                             |           |

4 或 8 字节

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

7. 处理NULL值为0
```sql
SELECT ProductName,UnitPrice*(UnitsInStock+IFNULL(UnitsOnOrder,0)) FROM Products
SELECT ProductName,UnitPrice*(UnitsInStock+COALESCE(UnitsOnOrder,0)) FROM Products
```

8. 数据类型

|    数据类型    |                             解析                             |
|--------------|--------------------------------------------------------------|
|CHAR(size)    |固定长度字符串（可包含字母、数字以及特殊字符）。size<255              |
|VARCHAR(size) |长度字符串（可包含字母、数字以及特殊字符）。size<255,大于255时转为text |
|TINYTEXT      |存放最大长度为 255 个字符的字符串。                                |
|TEXT          |存放最大长度为 65,535 个字符的字符串。                             |
|BLOB          |用于 BLOBs（Binary Large OBjects）。存放最多 65,535 字节的数据。   |
|MEDIUMTEXT    |存放最大长度为 16,777,215 个字符的字符串。                         |
|MEDIUMBLOB    |用于 BLOBs（Binary Large OBjects）。存放最多16,777,215 字节的数据。|
|LONGTEXT      |存放最大长度为 4,294,967,295 个字符的字符串。                      |
|LONGBLOB      |用于 BLOBs(Binary Large OBjects)。存放最多4,294,967,295字节的数据 |
|ENUM(x,y,z,etc.)|允许输入可能值的列表。可以在 ENUM 列表中列出最大 65535 个值。如果列表中不存在插入的值，则插入空值。这些值是按照输入的顺序排序的。|
|SET           |与 ENUM 类似，SET 最多只能包含64个列表项且SET可存储一个以上的选择。   |
|TINYINT(size) |带符号-128到127 ，无符号0到255。                                 |
|SMALLINT(size)|带符号范围-32768到32767，无符号0到65535, size 默认为 6。           |
|MEDIUMINT(size)|带符号范围-2^23到2^23-1，无符号的范围是0到2^24-1。 size 默认为9    |
|INT(size)     |带符号范围-2^31到2^31-1，无符号的范围是0到4294967295。siz 默认为11  |
|BIGINT(size)  |带符号的范围是-2^63到2^63-1，无符号的范围是0到2^64-1。size 默认为 20 |
|FLOAT(size,d) |带有浮动小数点的小数字。在 size 参数中规定显示最大位数。在 d 参数中规定小数点右侧的最大位数。|
|DOUBLE(size,d)|带有浮动小数点的大数字。在 size 参数中规显示定最大位数。在 d 参数中规定小数点右侧的最大位数。|
|DECIMAL(size,d)|作为字符串存储的 DOUBLE 类型，允许固定的小数点。在 size 参数中规定显示最大位数。在 d 参数中规定小数点右侧的最大位数。|
|DATE()        |日期。格式：YYYY-MM-DD,从 '1000-01-01' 到 '9999-12-31'          |
|DATETIME()    |日期和时间的组合。从'1000-01-01 00:00:00 到'9999-12-31 23:59:59' |
|TIMESTAMP()   |时间戳。TIMESTAMP 值使用 Unix 纪元('1970-01-01 00:00:00' UTC) 至今的秒数来存储。格式：YYYY-MM-DD HH:MM:SS，从 '1970-01-01 00:00:01' UTC 到 '2038-01-09 03:14:07' UTC|
|TIME()        |时间。格式：HH:MM:SS，从 '-838:59:59' 到 '838:59:59'            |
|YEAR()	       |2 位或 4 位格式的年。4 位格式所允许的值：1901 到 2155。2 位格式所允许的值：70 到 69，表示从 1970 到 2069。|

## 非关系型数据库

### MongoDB

|SQL术语/概念|MongoDB术语/概念|解释/说明                       |
|-----------|--------------|-------------------------------|
|database   |database      |数据库                          |
|table      |collection    |数据库表/集合                    |
|row        |document      |数据记录行/文档                   |
|column     |field         |数据字段/域                      |
|index      |index         |索引                            |
|table joins|              |表连接,MongoDB不支持             |
|primary key|primary key   |主键,MongoDB自动将_id字段设置为主键|