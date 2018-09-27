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

9. 获取一段时间内的数据
```sql
--一个月内
select createtime from user where DATE_SUB(CURDATE(), INTERVAL 1 MONTH) <= date(createtime);
--一周INTERVAL 7 DAY
```

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

9. 获取一段时间内的数据
```sql
--10min内
SELECT * FROM tablename WHERE 日期字段 > DATEADD(MINUTE,-10,GETDATE())
SELECT * FROM tablename WHERE datediff(mi,时间字段,getdate())<=10
```

## 非关系型数据库

### MongoDB

1. 安装与配置

* [初探MongoDB —— 介绍、安装和配置](https://www.cnblogs.com/jianglan/p/4423189.html)
* [MongoDB基本操作 —— 用Mongo.exe操作数据库增删改查](https://www.cnblogs.com/jianglan/p/4430299.html)
* [MongoDB可视化工具](https://robomongo.org/download)

2. 基础

|SQL术语/概念|MongoDB术语/概念|解释/说明                       |
|-----------|--------------|-------------------------------|
|database   |database      |数据库                          |
|table      |collection    |数据库表/集合                    |
|row        |document      |数据记录行/文档                   |
|column     |field         |数据字段/域                      |
|index      |index         |索引                            |
|table joins|              |表连接,MongoDB不支持             |
|primary key|primary key   |主键,MongoDB自动将_id字段设置为主键|

|数据类型       |描述                                                         |
|--------------|------------------------------------------------------------|
|String	字符串。|存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。|
|Integer       |整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。  |
|Boolean       |布尔值。用于存储布尔值（真/假）。                                |
|Double        |双精度浮点值。用于存储浮点值。                                   |
|Min/Max keys  |将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。        |
|Array         |用于将数组或列表或多个值存储为一个键。                            |
|Timestamp     |时间戳。记录文档修改或添加的具体时间。                            |
|Object        |用于内嵌文档。                                                |
|Null          |用于创建空值。                                                |
|Symbol        |符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。|
|Date          |日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 |Date 对象，传入年月日信息。|
|Object ID     |对象 ID。用于创建文档的 ID。                                    |
|Binary Data   |二进制数据。用于存储二进制数据。                                  |
|Code          |代码类型。用于在文档中存储 JavaScript 代码。                      |
|Regular expression|正则表达式类型。用于存储正则表达式。                          |


* ObjectId

ObjectId 类似唯一主键，可以很快的去生成和排序，包含 12 bytes：
1. 1-4表示创建 unix 时间戳,格林尼治时间 UTC 时间，比北京时间晚了 8 个小时
2. 5-7是机器标识码
3. 8-9由进程 id 组成 PID
4. 10-12是随机数

MongoDB 中存储的文档必须有一个 _id 键。这个键的值可以是任何类型，默认是 ObjectId 对象。

由于 ObjectId 中保存了创建的时间戳，所以不需要为文档保存时间戳字段，可通过 getTimestamp 函数来获取文档的创建时间:
```sql
var newObject = ObjectId()
newObject.getTimestamp()
--ISODate("2017-11-25T07:21:10Z")
newObject.str
--5a1919e63df83ce79df8b38f
```

* 字符串

BSON 字符串都是 UTF-8 编码。

* 时间戳

BSON 有一个特殊的时间戳类型用于 MongoDB 内部使用，与普通的 日期 类型不相关。 时间戳值是一个 64 位的值：
1. 前32位是一个 time_t 值（与Unix新纪元相差的秒数）
2. 后32位是在某秒中操作的一个递增的序数
3. 在单个 mongod 实例中，时间戳值通常是唯一的。

在复制集中， oplog 有一个 ts 字段。这个字段中的值使用BSON时间戳表示了操作时间。

* 日期

表示当前距离 Unix新纪元（1970年1月1日）的毫秒数。日期类型是有符号的, 负数表示 1970 年之前的日期。

```sql
var mydate1 = new Date()     --格林尼治时间
mydate1
--ISODate("2018-03-04T14:58:51.233Z")
typeof mydate1
--object
var mydate2 = ISODate() //格林尼治时间
mydate2
--ISODate("2018-03-04T15:00:45.479Z")
typeof mydate2
--object
--这样创建的时间是日期类型，可以使用 JS 中的 Date 类型的方法。

--返回一个时间类型的字符串：

var mydate1str = mydate1.toString()
mydate1str
--Sun Mar 04 2018 14:58:51 GMT+0000 (UTC) 
typeof mydate1str
--string
--或者
Date()
--Sun Mar 04 2018 15:02:59 GMT+0000 (UTC)   
```

3. 连接

标准 URI 连接:

mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
* mongodb:// 这是固定的格式，必须要指定。
* username:password@ 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登陆这个数据库
* host1 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。如果要连接复制集，请指定多个主机地址。
* portX 可选的指定端口，如果不填，默认为27017
* /database 如果指定username:password@，连接并验证登陆指定数据库。若不指定，默认打开 test 数据库。
* ?options 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开

样例：

* 连接本地数据库服务器，端口是默认的。

mongodb://localhost

* 使用用户名fred，密码foobar登录localhost的admin数据库。

mongodb://fred:foobar@localhost

* 使用用户名fred，密码foobar登录localhost的baz数据库。

mongodb://fred:foobar@localhost/baz

* 连接 replica pair, 服务器1为example1.com服务器2为example2。

mongodb://example1.com:27017,example2.com:27017

* 连接 replica set 三台服务器 (端口 27017, 27018, 和27019):

mongodb://localhost,localhost:27018,localhost:27019

* 连接 replica set 三台服务器, 写入操作应用在主服务器 并且分布查询到从服务器。

mongodb://host1,host2,host3/?slaveOk=true

* 直接连接第一个服务器，无论是replica set一部分或者主服务器或者从服务器。

mongodb://host1,host2,host3/?connect=direct;slaveOk=true

当你的连接服务器有优先级，还需要列出所有服务器，你可以使用上述连接方式。

* 安全模式连接到localhost:

mongodb://localhost/?safe=true

* 以安全模式连接到replica set，并且等待至少两个复制服务器成功写入，超时时间设置为2秒。

mongodb://host1,host2,host3/?safe=true;w=2;wtimeoutMS=2000

4. 语法

* 创建数据库
```sql
use DATABASE_NAME --创建数据库
show dbs --查看所有数据库
```

* 删除数据库
```sql
db.dropDatabase() --删除数据库
```

* 创建数据表(直接插入数据也能创建)
```sql
db.createCollection(name, options)
```

|字段        |类型|描述|
|-----------|----|---|
|capped     |布尔|（可选）如果为 true，则创建固定集合。<br>固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。<br>当该值为 true 时，必须指定 size 参数。|
|autoIndexId|布尔|（可选）如为 true，自动在 _id 字段创建索引。默认为 false。|
|size       |数值|（可选）为固定集合指定一个最大值（以字节计）。<br>如果 capped 为 true，也需要指定该字段。|
|max        |数值|（可选）指定固定集合中包含文档的最大数量。<br>在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。|

* 删除数据表
```sql
show collections --查看数据表
db.tablename.drop()  --删除数据表
```

* 插入数据
```sql
db.COLLECTION_NAME.insert(json) --插入数据
db.collection.insertOne({"a": 3}) --插入单条数据
db.collection.insertMany([{"b": 3}, {'c': 4}]) --插入多条数据
```

* 更新数据
```sql
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}}) --更新标题
db.col.save({
    "_id" : ObjectId("56064f89ade2f21f36b03136"),
    "title" : "MongoDB",
    "description" : "MongoDB 是一个 Nosql 数据库",
    "by" : "Runoob",
    "url" : "http://www.runoob.com",
    "tags" : [
            "mongodb",
            "NoSQL"
    ],
    "likes" : 110
}) --替换数据
db.col.update({"_id":"56064f89ade2f21f36b03136"}, {$unset:{ "test2" : "OK"}}) --移除键值对
```
* query : update的查询条件，类似sql update查询内where后面的。
* update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
* upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
* multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
* writeConcern :可选，抛出异常的级别。

* 删除数据
```sql
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
db.col.remove({'title':'MongoDB 教程'}) --删除指定title的所有数据
db.col.remove({}) --删除所有数据
db.inventory.deleteMany({}) --删除集合下全部文档
db.inventory.deleteMany({ status : "A" }) --删除 status 等于 A 的全部文档
db.inventory.deleteOne( { status: "D" } ) --删除 status 等于 D 的一个文档
```

* query :（可选）删除的文档的条件。
* justOne : （可选）如果设为 true 或 1，则只删除一个文档。
* writeConcern :（可选）抛出异常的级别。

---

* 查询数据
```sql
db.collection.find(query, projection)
db.col.find().pretty() --以易读的方式来读取数据
db.col.find({key1:value1, key2:value2}).pretty() --and
db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty() --or
db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty() --混合
--若不指定 projection，则默认返回所有键，指定 projection 格式如下，有两种模式
db.collection.find(query, {title: 1, by: 1}) --inclusion模式 指定返回的键，不返回其他键
db.collection.find(query, {title: 0, by: 0}) --exclusion模式 指定不返回的键,返回其他键
--_id 键默认返回，需要主动指定 _id:0 才会隐藏
--两种模式不可混用（因为这样的话无法推断其他键是否应返回）
db.collection.find(query, {title: 1, by: 0}) --错误
--只能全1或全0，除了在inclusion模式时可以指定_id为0
db.collection.find(query, {_id:0, title: 1, by: 1}) --正确
--若不想指定查询条件参数 query 可以 用 {} 代替，但是需要指定 projection 参数：
querydb.collection.find({}, {title: 1})
```

* query ：可选，使用查询操作符指定查询条件
* projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

条件操作符

* db.col.find({"by":"菜鸟教程"}).pretty()

where by = '菜鸟教程'

* db.col.find({"likes":{$lt:50}}).pretty()

where likes < 50

* db.col.find({"likes":{$lte:50}}).pretty()

where likes <= 50

* db.col.find({"likes":{$gt:50}}).pretty()

where likes > 50

* db.col.find({"likes":{$gte:50}}).pretty()

where likes >= 50

* db.col.find({"likes":{$ne:50}}).pretty()

where likes != 50

* 查询 title 包含"教"字的文档：

db.col.find({title:/教/})

* 查询 title 字段以"教"字开头的文档：

db.col.find({title:/^教/})

* 查询 titl e字段以"教"字结尾的文档：

db.col.find({title:/教$/})

---

* $type 操作符 (类型匹配)

|类型	             |数字   |备注           |
|-------------------|-------|--------------|
|Double             |1      |              |
|String             |2      |              |
|Object             |3      |              |
|Array              |4      |              |
|Binary data        |5      |              |
|Undefined          |6      |已废弃。       |
|Object id          |7      |              |
|Boolean            |8      |              |
|Date               |9      |              |
|Null               |10     |              |
|Regular Expression |11     |              |
|JavaScript         |13     |              |
|Symbol             |14     |              |
|JavaScript(with scope)|15  |              |
|32-bit integer     |16     |              |
|Timestamp          |17     |              |
|64-bit integer     |18     |              |
|Min key            |255	|Query with -1.|
|Max key            |127	|              |

```sql
db.col.find({"title" : {$type : 2}})
db.col.find({"title" : {$type : 'string'}})
```

* limit与skip

limit(n) 是用来规定显示的条数，而 skip(n) 是用来在符合条件的记录中从第一个记录跳过的条数，这两个函数可以交换使用。
```sql
--读取指定数量的数据,跳过指定数量的数据
db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
--相当于mysql 中limit (10,100)
db.COLLECTION_NAME.find().skip(10).limit(100)
```

* sort排序

```sql
--1 为升序，-1 为降序排列。
db.COLLECTION_NAME.find().sort({KEY:1})
```
skip(), limilt(), sort()三个放在一起执行的时候，执行的顺序是先 sort(), 然后是 skip()，最后是显示的 limit()

* 索引
1. 创建索引
```sql
db.collection.createIndex(keys, options)
--1按升序创建索引，-1按降序创建索引
db.col.createIndex({"title":1,"description":-1})
db.values.createIndex({open: 1, close: 1}, {background: true})
```

|参数               |类型         |描述                   |
|------------------|-------------|----------------------|
|background        |Boolean      |建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为false。|
|unique            |Boolean      |建立的索引是否唯一。指定为true创建唯一索引。默认值为false.|
|name              |string       |索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。|
|dropDups          |Boolean      |3.0+版本已废弃。在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 false.|
|sparse            |Boolean      |对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 false.|
|expireAfterSeconds|integer      |指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。|
|v                 |index version|索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。|
|weights           |document     |索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。|
|default_language  |string       |对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语|
|language_override |string       |对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language.|

2. 覆盖索引查询(不从数据库找数据)
```sql
--创建索引（旧版本写法）
db.users.ensureIndex({gender:1,user_name:1})
--查询，它会从索引中提取数据，这是非常快速的数据查询
--由于我们的索引中不包括 _id 字段，_id在查询中会默认返回，我们可以在MongoDB的查询结果集中排除它。
db.users.find({gender:"M"},{user_name:1,_id:0})
--查询就不会被覆盖，从数据库查数据
db.users.find({gender:"M"},{user_name:1})
```
以下的查询，不能使用覆盖索引查询：
1. 所有索引字段是一个数组
2. 所有索引字段是一个子文档
---
3. 索引子文档字段

```sql
{
   "address": {
      "city": "Los Angeles",
      "state": "California",
      "pincode": "123"
   },
   "tags": [
      "music",
      "cricket",
      "blogs"
   ],
   "name": "Tom Benzamin"
}
--在我们为数组 tags 创建索引时，会为 music、cricket、blogs三个值建立单独的索引。
db.users.ensureIndex({"tags":1})
db.users.find({tags:"cricket"})
--索引子文档字段
db.users.ensureIndex({"address.city":1,"address.state":1,"address.pincode":1})
db.users.find({"address.city":"Los Angeles"})   
db.users.find({"address.state":"California","address.city":"Los Angeles"}) 
db.users.find({"address.city":"Los Angeles","address.state":"California","address.pincode":"123"})
```

4. 索引限制
* 查询限制

索引不能被以下的查询使用：
1. 正则表达式及非操作符，如 $nin, $not, 等。
2. 算术运算符，如 $mod, 等。
3. $where 子句

* 索引键限制
从2.6版本开始，如果现有的索引字段的值超过索引键的限制，MongoDB中不会创建索引。

* 插入文档超过索引键限制
如果文档的索引字段值超过了索引键的限制，MongoDB不会将任何文档转换成索引的集合。

* 最大范围
1. 集合中索引不能超过64个
2. 索引名的长度不能超过128个字符
3. 一个复合索引最多可以有31个字段
---
* aggregate聚合

用于处理数据(如统计平均值,求和，聚类等)，并返回计算后的数据结果。类似sql语句中的 count和group by。

```sql
-- by_user 字段对数据进行分组，并计算 by_user 字段相同值的总和
 db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
select by_user as _id, count(*) as num_tutorial from mycol group by by_user
```

|表达式    |描述                                   |实例                |
|---------|--------------------------------------|-------------------|
|$sum     |计算总和。                              |db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}])|
|$avg     |计算平均值                              |db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}])|
|$min     |获取集合中所有文档对应值得最小值。          |db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}])|
|$max     |获取集合中所有文档对应值得最大值。          |db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}])|
|$push    |在结果文档中插入值到一个数组中。            |db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}])|
|$addToSet|在结果文档中插入值到一个数组中，但不创建副本。|db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}])|
|$first   |根据资源文档的排序获取第一个文档数据。       |db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}])|
|$last    |根据资源文档的排序获取最后一个文档数据。     |db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}])|

* 管道

MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

* $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
* $match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
* $limit：用来限制MongoDB聚合管道返回的文档数。
* $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
* $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
* $group：将集合中的文档分组，可用于统计结果。
* $sort：将输入文档排序后输出。
* $geoNear：输出接近某一地理位置的有序文档。

```sql
db.article.aggregate(
    { $project : {
        title : 1 ,
        author : 1 ,
    }}
 );
--结果中还有_id,tilte和author三个字段，默认情况下_id字段是被包含的，如果不包含_id可以:
db.article.aggregate(
    { $project : {
        _id : 0 ,
        title : 1 ,
        author : 1
    }});
--$match用于获取分数大于70小于或等于90记录，然后将符合条件的记录送到下一阶段$group管道操作符进行处理。
db.articles.aggregate( [
                        { $match : { score : { $gt : 70, $lte : 90 } } },
                        { $group: { _id: null, count: { $sum: 1 } } }
                       ] );
--经过$skip管道操作符处理后，前五个文档被"过滤"掉。
db.article.aggregate(
    { $skip : 5 });
--按日、按月、按年、按周、按小时、按分钟聚合操作如下
db.getCollection('m_msg_tb').aggregate(
[
    {$match:{m_id:10001,mark_time:{$gt:new Date(2017,8,0)}}},
    {$group: {
       _id: {$dayOfMonth:'$mark_time'},
        pv: {$sum: 1}
        }
    },
    {$sort: {"_id": 1}}
])
```

时间关键字：
 * $dayOfYear: 返回该日期是这一年的第几天（全年 366 天）。
 * $dayOfMonth: 返回该日期是这一个月的第几天（1到31）。
 * $dayOfWeek: 返回的是这个周的星期几（1：星期日，7：星期六）。
 * $year: 返回该日期的年份部分。
 * $month： 返回该日期的月份部分（ 1 到 12）。
 * $week： 返回该日期是所在年的第几个星期（ 0 到 53）。
 * $hour： 返回该日期的小时部分。
 * $minute: 返回该日期的分钟部分。
 * $second: 返回该日期的秒部分（以0到59之间的数字形式返回日期的第二部分，但可以是60来计算闰秒）。
 * $millisecond：返回该日期的毫秒部分（0 到 999）。
 * $dateToString： { $dateToString: { format: , date: } }。

5. 技术与原理

* 复制（副本集）

用途：
* 保障数据的安全性
* 数据高可用性 (24*7)
* 灾难恢复
* 无需停机维护（如备份，重建索引，压缩）
* 分布式读取数据

启动服务器：

mongod --port "PORT" --dbpath "YOUR_DB_DATA_PATH" --replSet "REPLICA_SET_INSTANCE_NAME"

样例：

mongod --port 27017 --dbpath "D:\set up\mongodb\data" --replSet rs0

以上实例会启动一个名为rs0的MongoDB实例，其端口号为27017。

启动后打开命令提示框并连接上mongoDB服务。

在Mongo客户端使用命令rs.initiate()来启动一个新的副本集。

我们可以使用rs.conf()来查看副本集的配置

查看副本集状态使用 rs.status() 命令

添加副本集的成员，我们需要使用多台服务器来启动mongo服务。进入Mongo客户端，并使用rs.add()方法来添加副本集的成员。

rs.add("mongod1.net:27017")

MongoDB中你只能通过主节点将Mongo服务添加到副本集中， 判断当前运行的Mongo服务是否为主节点可以使用命令db.isMaster() 。

MongoDB的副本集与我们常见的主从有所不同，主从在主机宕机后所有服务将停止，而副本集在主机宕机后，副本会接管主节点成为主节点，不会出现宕机的情况。

* 分片

当MongoDB存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据。
```text
分片结构端口分布如下：

Shard Server 1：27020
Shard Server 2：27021
Shard Server 3：27022
Shard Server 4：27023
Config Server ：27100
Route Process：40000
步骤一：启动Shard Server

[root@100 /]# mkdir -p /www/mongoDB/shard/s0
[root@100 /]# mkdir -p /www/mongoDB/shard/s1
[root@100 /]# mkdir -p /www/mongoDB/shard/s2
[root@100 /]# mkdir -p /www/mongoDB/shard/s3
[root@100 /]# mkdir -p /www/mongoDB/shard/log
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 27020 --dbpath=/www/mongoDB/shard/s0 --logpath=/www/mongoDB/shard/log/s0.log --logappend --fork
....
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 27023 --dbpath=/www/mongoDB/shard/s3 --logpath=/www/mongoDB/shard/log/s3.log --logappend --fork
步骤二： 启动Config Server

[root@100 /]# mkdir -p /www/mongoDB/shard/config
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 27100 --dbpath=/www/mongoDB/shard/config --logpath=/www/mongoDB/shard/log/config.log --logappend --fork
注意：这里我们完全可以像启动普通mongodb服务一样启动，不需要添加—shardsvr和configsvr参数。因为这两个参数的作用就是改变启动端口的，所以我们自行指定了端口就可以。

步骤三： 启动Route Process

/usr/local/mongoDB/bin/mongos --port 40000 --configdb localhost:27100 --fork --logpath=/www/mongoDB/shard/log/route.log --chunkSize 500
mongos启动参数中，chunkSize这一项是用来指定chunk的大小的，单位是MB，默认大小为200MB.

步骤四： 配置Sharding

接下来，我们使用MongoDB Shell登录到mongos，添加Shard节点

[root@100 shard]# /usr/local/mongoDB/bin/mongo admin --port 40000
MongoDB shell version: 2.0.7
connecting to: 127.0.0.1:40000/admin
mongos> db.runCommand({ addshard:"localhost:27020" })
{ "shardAdded" : "shard0000", "ok" : 1 }
......
mongos> db.runCommand({ addshard:"localhost:27029" })
{ "shardAdded" : "shard0009", "ok" : 1 }
mongos> db.runCommand({ enablesharding:"test" }) #设置分片存储的数据库
{ "ok" : 1 }
mongos> db.runCommand({ shardcollection: "test.log", key: { id:1,time:1}})
{ "collectionsharded" : "test.log", "ok" : 1 }
步骤五： 程序代码内无需太大更改，直接按照连接普通的mongo数据库那样，将数据库连接接入接口40000

---

1. 创建Sharding复制集 rs0

# mkdir /data/log
# mkdir /data/db1
# nohup mongod --port 27020 --dbpath=/data/db1 --logpath=/data/log/rs0-1.log --logappend --fork --shardsvr --replSet=rs0 &

# mkdir /data/db2
# nohup mongod --port 27021 --dbpath=/data/db2 --logpath=/data/log/rs0-2.log --logappend --fork --shardsvr --replSet=rs0 &
1.1 复制集rs0配置

# mongo localhost:27020 > rs.initiate({_id: 'rs0', members: [{_id: 0, host: 'localhost:27020'}, {_id: 1, host: 'localhost:27021'}]}) > rs.isMaster() #查看主从关系
2. 创建Sharding复制集 rs1

# mkdir /data/db3
# nohup mongod --port 27030 --dbpath=/data/db3 --logpath=/data/log/rs1-1.log --logappend --fork --shardsvr --replSet=rs1 &
# mkdir /data/db4
# nohup mongod --port 27031 --dbpath=/data/db4 --logpath=/data/log/rs1-2.log --logappend --fork --shardsvr --replSet=rs1 &
2.1 复制集rs1配置

# mongo localhost:27030
> rs.initiate({_id: 'rs1', members: [{_id: 0, host: 'localhost:27030'}, {_id: 1, host: 'localhost:27031'}]})
> rs.isMaster() #查看主从关系
3. 创建Config复制集 conf

# mkdir /data/conf1
# nohup mongod --port 27100 --dbpath=/data/conf1 --logpath=/data/log/conf-1.log --logappend --fork --configsvr --replSet=conf &
# mkdir /data/conf2
# nohup mongod --port 27101 --dbpath=/data/conf2 --logpath=/data/log/conf-2.log --logappend --fork --configsvr --replSet=conf &
3.1 复制集conf配置

# mongo localhost:27100
> rs.initiate({_id: 'conf', members: [{_id: 0, host: 'localhost:27100'}, {_id: 1, host: 'localhost:27101'}]})
> rs.isMaster() #查看主从关系
4. 创建Route

# nohup mongos --port 40000 --configdb conf/localhost:27100,localhost:27101 --fork --logpath=/data/log/route.log --logappend & 
4.1 设置分片

# mongo localhost:40000
> use admin
> db.runCommand({ addshard: 'rs0/localhost:27020,localhost:27021'})
> db.runCommand({ addshard: 'rs1/localhost:27030,localhost:27031'})
> db.runCommand({ enablesharding: 'test'})
> db.runCommand({ shardcollection: 'test.user', key: {name: 1}})
```

* 备份(mongodump)与恢复(mongorestore)

1. 备份:

mongodump -h dbhost -d dbname -o dbdirectory

* -h：
MongDB所在服务器地址，例如：127.0.0.1，当然也可以指定端口号：127.0.0.1:27017

* -d：
需要备份的数据库实例，例如：test

* -o：
备份的数据存放位置，例如：c:\data\dump，当然该目录需要提前建立，在备份完成后，系统自动在dump目录下建立一个test目录，这个目录里面存放该数据库实例的备份数据。

mongodump 命令可选参数列表如下所示：

|语法	                                       |描述	                     |实例|
|---------------------------------------------|-----------------------------|----|
|mongodump --host HOST_NAME --port PORT_NUMBER    |该命令将备份所有MongoDB数据 |mongodump --host runoob.com --port 27017|
|mongodump --dbpath DB_PATH --out BACKUP_DIRECTORY|                         |mongodump --dbpath /data/db/ --out /data/backup/|
|mongodump --collection COLLECTION --db DB_NAME   |该命令将备份指定数据库的集合。|mongodump --collection mycol --db test|

2. 恢复

mongorestore -h \<hostname\><:port> -d dbname \<path\>

* --host <:port>, -h <:port>：
MongoDB所在服务器地址，默认为： localhost:27017

* --db , -d ：
需要恢复的数据库实例，例如：test，当然这个名称也可以和备份时候的不一样，比如test2

* --drop：
恢复的时候，先删除当前数据，然后恢复备份的数据。就是说，恢复后，备份后添加修改的数据都会被删除，慎用哦！

* \<path\>：
mongorestore 最后的一个参数，设置备份数据所在位置，例如：c:\data\dump\test。

你不能同时指定 \<path\> 和 --dir 选项，--dir也可以设置备份目录。

* --dir：
指定备份的目录

你不能同时指定 \<path\> 和 --dir 选项。

---

* 监控

mongostat是mongodb自带的状态检测工具，在命令行下使用。它会间隔固定时间获取mongodb的当前运行状态，并输出。如果你发现数据库突然变慢或者有其他问题的话，你第一手的操作就考虑采用mongostat来查看mongo的状态。

D:\set up\mongodb\bin>mongostat

mongotop也是mongodb下的一个内置工具，mongotop提供了一个方法，用来跟踪一个MongoDB的实例，查看哪些大量的时间花费在读取和写入数据。 mongotop提供每个集合的水平的统计数据。默认情况下，mongotop返回值的每一秒。

D:\set up\mongodb\bin>mongotop

E:\mongodb-win32-x86_64-2.2.1\bin>mongotop 10

后面的10是<sleeptime>参数 ，可以不使用，等待的时间长度，以秒为单位，mongotop等待调用之间。通过的默认mongotop返回数据的每一秒。

 E:\mongodb-win32-x86_64-2.2.1\bin>mongotop --locks

输出结果字段说明：

* ns：
包含数据库命名空间，后者结合了数据库名称和集合。

* db：
包含数据库的名称。名为 . 的数据库针对全局锁定，而非特定数据库。

* total：
mongod花费的时间工作在这个命名空间提供总额。

* read：
提供了大量的时间，这mongod花费在执行读操作，在此命名空间。

* write：
提供这个命名空间进行写操作，这mongod花了大量的时间。

6. 高级应用

* 关系

1. 嵌入式关系

保存在单一的文档中，可以比较容易的获取和维护数据
```text
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin",
   "address": [
      {
         "building": "22 A, Indiana Apt",
         "pincode": 123456,
         "city": "Los Angeles",
         "state": "California"
      },
      {
         "building": "170 A, Acropolis Apt",
         "pincode": 456789,
         "city": "Chicago",
         "state": "Illinois"
      }]
} 
```
查询方法
```sql
db.users.findOne({"name":"Tom Benzamin"},{"address":1})
```

2. 引用式关系

把用户数据文档和用户地址数据文档分开，通过引用文档的 id 字段来建立关系。
```text
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin",
   "address_ids": [
      ObjectId("52ffc4a5d85242602e000000"),
      ObjectId("52ffc4a5d85242602e000001")
   ]
}
```
查询方法
```sql
var result = db.users.findOne({"name":"Tom Benzamin"},{"address_ids":1})
var addresses = db.address.find({"_id":{"$in":result["address_ids"]}})
```

引用方式：

DBRefs
* $ref：集合名称
* $id：引用的id
* $db:数据库名称，可选参数
```text
{
   "_id":ObjectId("53402597d852426020000002"),
   "address": {
   "$ref": "address_home",
   "$id": ObjectId("534009e4d852427820000002"),
   "$db": "runoob"},
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin"
}
```
查询方法
```sql
var user = db.users.findOne({"name":"Tom Benzamin"})
var dbRef = user.address
db[dbRef.$ref].findOne({"_id":(dbRef.$id)})
```

* 查询分析
```sql
db.users.find({gender:"M"},{user_name:1,_id:0}).explain()
db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1})
db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1}).explain()
```
* indexOnly: 字段为 true ，表示我们使用了索引。
* cursor：因为这个查询使用了索引，MongoDB 中索引存储在B树结构中，所以这是也使用了 BtreeCursor 类型的游标。如果没有使用索引，游标的类型是 BasicCursor。这个键还会给出你所使用的索引的名称，你通过这个名称可以查看当前数据库下的system.indexes集合（系统自动创建，由于存储索引信息，这个稍微会提到）来得到索引的详细信息。
* n：当前查询返回的文档数量。
* nscanned/nscannedObjects：表明当前这次查询一共扫描了集合中多少个文档，我们的目的是，让这个数值和返回文档的数量越接近越好。
* millis：当前查询所需时间，毫秒数。
* indexBounds：当前查询具体使用的索引。
---
* 原子操作
```sql
--判断是否可结算并更新新的结算信息
db.books.findAndModify ( {
   query: {
            _id: 123456789,
            available: { $gt: 0 }
          },
   update: {
             $inc: { available: -1 },
             $push: { checkout: { by: "abc", date: new Date() } }
           }
} )
```

* $set
```sql
用来指定一个键并更新键值，若键不存在并创建。
{ $set : { field : value } }
```
* $unset
```sql
用来删除一个键。
{ $unset : { field : 1} }
```
* $inc
```sql
$inc可以对文档的某个值为数字型（只能为满足要求的数字）的键进行增减的操作。
{ $inc : { field : value } }
```
* $push
```sql
--把value追加到field里面去，field一定要是数组类型才行，如果field不存在，会新增一个数组类型加进去。
{ $push : { field : value } }
```
* $pushAll
```sql
--同$push,只是一次可以追加多个值到一个数组字段内。
{ $pushAll : { field : value_array } }
```
* $pull
```sql
--从数组field内删除一个等于value值。
{ $pull : { field : _value } }
```
* $addToSet
```
增加一个值到数组内，而且只有当这个值不在数组内才增加。
```
* $pop
```sql
--删除数组的第一个或最后一个元素
{ $pop : { field : 1 } }
```
* $rename
```sql
--修改字段名称
{ $rename : { old_field_name : new_field_name } }
```
* $bit
```sql
--位操作，integer类型
{$bit : { field : {and : 5}}}
```
* 偏移操作符
```sql
t.find() { "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC", "comments" : [ { "by" : "joe", "votes" : 3 }, { "by" : "jane", "votes" : 7 } ] }
 
t.update( {'comments.by':'joe'}, {$inc:{'comments.$.votes':1}}, false, true )
 
t.find() { "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC", "comments" : [ { "by" : "joe", "votes" : 4 }, { "by" : "jane", "votes" : 7 } ] }
```
---
* Map Reduce 大型聚合查询

Map-Reduce是一种计算模型，将大批量的工作（数据）分解（MAP）执行，然后再将结果合并成最终结果（REDUCE）。

```sql
db.collection.mapReduce(
   function() {emit(key,value);},  //map 函数
   function(key,values) {return reduceFunction},   //reduce 函数
   {
      out: collection,
      query: document,
      sort: document,
      limit: number
   }
)
```
参数说明:

* map ：映射函数 (生成键值对序列,作为 reduce 函数参数)。
* reduce 统计函数，reduce函数的任务就是将key-values变成key-value，也就是把values数组变成一个单一的值value。
* out 统计结果存放集合 (不指定则使用临时集合,在客户端断开后自动删除)。
* query 一个筛选条件，只有满足条件的文档才会调用map函数。（query。limit，sort可以随意组合）
* sort 和limit结合的sort排序参数（也是在发往map函数前给文档排序），可以优化分组机制
* limit 发往map函数的文档数量的上限（要是没有limit，单独使用sort的用处不大）

![Map Reduce](map-reduce.bakedsvg.svg)

---
* 全文检索

1. 启用

MongoDB 在 2.6 版本以后是默认开启全文检索的，如果使用之前的版本，需要使用启用全文检索:
```
db.adminCommand({setParameter:true,textSearchEnabled:true})
或
mongod --setParameter textSearchEnabled=true
```

2. 创建
```
{
   "post_text": "enjoy the mongodb articles on Runoob",
   "tags": [
      "mongodb",
      "runoob"
   ]
}
```
```sql
db.posts.ensureIndex({post_text:"text"})
```

3. 使用
```sql
db.posts.find({$text:{$search:"runoob"}})
或
db.posts.runCommand("text",{search:"runoob"})
```

4. 删除
```sql
db.posts.getIndexes() --查找索引名
db.posts.dropIndex("post_text_text") 
```

* 正则表达式
```
{
   "post_text": "enjoy the mongodb articles on Runoob",
   "tags": [
      "mongodb",
      "runoob"
   ]
}
```
```sql
db.posts.find({post_text:{$regex:"runoob"}})
db.posts.find({post_text:/runoob/})
db.posts.find({post_text:{$regex:"runoob",$options:"$i"}}) -- 相当于/runoob/i忽略大小写
db.posts.find({tags:{$regex:"run"}})
```
$regex操作符的使用:

$regex操作符中的option选项可以改变正则匹配的默认行为，它包括i, m, x以及S四个选项，其含义如下

* i 忽略大小写，{<field>{$regex/pattern/i}}，设置i选项后，模式中的字母会进行大小写不敏感匹配。
* m 多行匹配模式，{<field>{$regex/pattern/,$options:'m'}，m选项会更改^和$元字符的默认行为，分别使用与行的开头和结尾匹配，而不是与输入字符串的开头和结尾匹配。
* x 忽略非转义的空白字符，{<field>:{$regex:/pattern/,$options:'m'}，设置x选项后，正则表达式中的非转义的空白字符将被忽略，同时井号(#)被解释为注释的开头注，只能显式位于option选项中。
* s 单行匹配模式{<field>:{$regex:/pattern/,$options:'s'}，设置s选项后，会改变模式中的点号(.)元字符的默认行为，它会匹配所有字符，包括换行符(\n)，只能显式位于option选项中。
使用$regex操作符时，需要注意下面几个问题:

* i，m，x，s可以组合使用，例如:{name:{$regex:/j*k/,$options:"si"}}
在设置索弓}的字段上进行正则匹配可以提高查询速度，而且当正则表达式使用的是前缀表达式时，查询速度会进一步提高，例如:{name:{$regex: /^joe/}

---
* GridFS

GridFS 用于存储和恢复那些超过16M（BSON文件限制）的文件(如：图片、音频、视频等)。

GridFS 也是文件存储的一种方式，但是它是存储在MonoDB的集合中。

GridFS 可以更好的存储大于16M的文件。

GridFS 会将大文件对象分割成多个小的chunk(文件片段),一般为256k/个,每个chunk将作为MongoDB的一个文档(document)被存储在chunks集合中。

GridFS 用两个集合来存储一个文件：fs.files与fs.chunks。

每个文件的实际内容被存在chunks(二进制数据)中,和文件有关的meta数据(filename,content_type,还有用户自定义的属性)将会被存在files集合中。

fs.files:
```
{
   "filename": "test.txt",
   "chunkSize": NumberInt(261120),
   "uploadDate": ISODate("2014-04-13T11:32:33.557Z"),
   "md5": "7b762939321e146569b07f72c62cca4f",
   "length": NumberInt(646)
}
```
fs.chunks:
```
{
   "files_id": ObjectId("534a75d19f54bfec8a2fe44b"),
   "n": NumberInt(0),
   "data": "Mongo Binary Data"
}
```

添加文件:

put 命令来存储 mp3 文件

```
mongofiles.exe -d gridfs put song.mp3
```

查看文档:
```sql
db.fs.files.find()
```

获取chunk数据
```
db.fs.chunks.find({files_id:ObjectId('534a811bf8b4aa4d33fdf94d')})
```

* 固定集合（Capped Collections）

固定集合是性能出色且有着固定大小的集合，就像一个环形队列，当集合空间用完后，再插入的元素就会覆盖最初始的头部的元素

属性
1. 对固定集合进行插入速度极快
2. 按照插入顺序的查询输出速度极快
3. 能够在插入最新数据时,淘汰最早的数据

用法
1. 储存日志信息
2. 缓存一些少量的文档

创建:
```sql
--指定文档个数1000,可不指定,大小10000kb
db.createCollection("cappedLogCollection",{capped:true,size:10000,max:1000})
```

判断集合是否为固定集合:
```sql
db.cappedLogCollection.isCapped()
```

将已存在的集合转换为固定集合:
```sql
db.runCommand({"convertToCapped":"posts",size:10000})
```

查询:

默认情况下查询就是按照插入顺序返回的,也可以使用$natural调整返回顺序
```sql
db.cappedLogCollection.find().sort({$natural:-1})
```

* _id 自动增长

使用 counters 集合
```sql
db.createCollection("counters")
db.counters.insert({_id:"productid",sequence_value:0})
function getNextSequenceValue(sequenceName){
   var sequenceDocument = db.counters.findAndModify(
      {
         query:{_id: sequenceName },
         update: {$inc:{sequence_value:1}},
         "new":true
      });
   return sequenceDocument.sequence_value;
}
db.products.insert({
   "_id":getNextSequenceValue("productid"),
   "product_name":"Apple iPhone",
   "category":"mobiles"})

db.products.insert({
   "_id":getNextSequenceValue("productid"),
   "product_name":"Samsung S3",
   "category":"mobiles"})
db.products.find()
```













