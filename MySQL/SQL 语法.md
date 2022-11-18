# 1、基础

模式定义了数据如何存储、存储什么样的数据以及数据如何分解等信息，数据库和表都有模式。

主键的值不允许修改，也不允许复用（不能将已经删除的主键值赋给新数据行的主键）。

SQL（Structured Query Language)，标准 SQL 由 ANSI 标准委员会管理，从而称为 ANSI SQL。各个 DBMS 都有自己的实现，如 PL/SQL、Transact-SQL 等。

SQL 语句不区分大小写，但是数据库表名、列名和值是否区分依赖于具体的数据库管理系统 DBMS (Database Management System)以及配置。

SQL 支持以下三种注释：

```sql
## 注释
SELECT *
FROM mytable; -- 注释
/* 注释1
   注释2 */
```

数据库创建与使用：

```sql
CREATE DATABASE test;
USE test;
```

> **接下来的主要内容均为表操作。**

# 2、创建表

```sql
CREATE TABLE mytable (
  # int 类型，不为空，自增
  id INT NOT NULL AUTO_INCREMENT,
  # int 类型，不可为空，默认值为 1，不为空
  col1 INT NOT NULL DEFAULT 1,
  # 变长字符串类型，最长为 45 个字符，可以为空
  col2 VARCHAR(45) NULL,
  # 日期类型，可为空
  col3 DATE NULL,
  # 设置主键为 id
  PRIMARY KEY (`id`));
```

# 3、修改表

添加列

```sql
ALTER TABLE mytable
ADD col CHAR(20);
```

修改列

```sql
ALTER TABLE mytable
DROP COLUMN col;
```

删除表

```sql
DROP TABLE mytable;
```

`ALTER` 用法：

```sql
ALTER TABLE <表名> [修改选项] 
-- 修改选项可以为：
{ ADD COLUMN <列名> <类型>
| CHANGE COLUMN <旧列名> <新列名> <新列类型>
| ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT }
| MODIFY COLUMN <列名> <类型>
| DROP COLUMN <列名>
| RENAME TO <新表名> }
```



# 4、插入

向表中插入数据

```sql
INSERT INTO mytable(col1, col2)
VALUES(val1, val2);
```

插入检索出来的数据

```sql
INSERT INTO mytable1(col1, col2)
SELECT col1, col2
FROM mytable2;
```

将一个表中的所有内容复制到另一个表

```sql
CREATE TABLE newtable AS
SELECT * FROM mytable;
```

除了 `INSERT` 语句外，还可采用 `SELECT INTO` 语句。`INSERT INTO` 用于向表中插入行，而 `SELECT INTO` 语句用于从一个表中选择出数据后，将此数据插入到另一个表中。

# 5、更新

```sql
UPDATE mytable
SET col = val
WHERE id = 1;
```

# 6、删除

```sql
DELETE FROM mytable
WHERE id = 1;
```

TRUNCATE TABLE + 表明，可清空表中的所有内容。

```sql
TRUNCATE TABLE mytable;
```

使用更新和删除操作时一定要用 WHERE 子句，不然会把整张表的数据都破坏。

> 多表连接删除

```sql
DELETE p1 FROM person AS p1, person AS p2 WHERE p1.`Id` > p2.`Id`;
```

这种属于自连接删除，仅保留表中最小的那行数据。

# 7、查询

## DISTINCT

选择出来的相同的内容只会出现一次，自动排除重复元素，这里的重复指的是所有内容相同相同才算重复。

```sql
SELECT DISTINCT col1, col2
FROM mytable;
```

## LIMIT

限制返回的行数。可以有两个参数，第一个参数为起始行，从 0 开始；第二个参数为返回的总行数。

```sql
-- 返回前 5 行
SELECT *
FROM mytable
LIMIT 5;
```

```sql
-- 从第 2 行开始，返回 2 ~ 6 行
SELECT *
FROM mytable
LIMIT 1, 5;
```

```sql
-- 从第 3 行开始，返回 3 ~ 5 行
SELECT *
FROM mytable
LIMIT 2, 3;
```

## UNION

使用 UNION 可对多个查询结果进行连接

- UNION必须由两条以上的SELECT语句组成，语句之间用关键字UNION分割。
- UNION中的每个查询必须包含相同的列、表达式或聚集函数（各个列不需要以相同的次序列出）。
- 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型。
- 如果取出来的数据不需要去重，使用UNION ALL。

# 8、排序

- **ASC** ：升序（默认）
- **DESC** ：降序

可以按多个列进行排序，并且为每个列指定不同的排序方式：

```sql
SELECT *
FROM mytable
ORDER BY col1 DESC, col2 ASC;
```

# 9、过滤

不进行过滤的数据非常大，导致通过网络传输了多余的数据，从而浪费了网络带宽。因此尽量使用 SQL 语句来过滤不必要的数据，而不是传输所有的数据到客户端中然后由客户端进行过滤。

在 SQL 语句中，使用 where 进行过滤

```sql
SELECT * FROM mytable WHERE col1 != 1;
```

where 可使用的操作符

| 操作符  |     说明     |
| :-----: | :----------: |
|    =    |     等于     |
|    <    |     小于     |
|    >    |     大于     |
| <>  !=  |    不等于    |
|  <= !>  |   小于等于   |
|  >= !<  |   大于等于   |
| BETWEEN | 在两个值之间 |
| IS NULL |  为 NULL 值  |

注意：null，0，空字符三者不同。

**AND** 和 **OR** 用来连接多个条件，优先处理 AND，当一个过滤表达式涉及到多个 AND 和 OR 时，可以使用 () 来决定优先级，使得优先级关系更清晰。

**IN** 操作符用于匹配一组值，其后也可以接一个 SELECT 子句，从而匹配子查询得到的一组值。

```sql
SELECT * FROM mytable WHERE col1 IN (1, 2);
SELECT * FROM mytable WHERE col1 IN (SELECT col1 FROM mytable2);
```

注意，在 IN 后面接 SELECT 子句时，需要加括号

**NOT** 操作符用于否定一个条件。

```sql
SELECT * FROM mytable WHERE col1 NOT IN (1, 2);
```

# 10、通配符

通配符也是用在 WHERE 过滤语句中，但只能用于文本字段，使用 LIKE 进行通配符匹配

- %：\>= 0 个任意字符
- _：== 1 个任意字符
- [ ]：可以匹配集合内的字符
- [^ ]：不匹配集合内的字符，例如 [ ^ ab] 表示不匹配 a 或 b

```sql
SELECT * FROM mytable WHERE col2 LIKE "a%";
```

不要滥用通配符，通配符位于开头处匹配会非常慢。

# 11、计算字段

在数据库服务器上完成数据的转换和格式化的工作往往比客户端上快得多，因此通常在数据库服务器上进行字段的格式转换。

```sql
SELECT col1 AS c1 FROM mytable;
```

**CONCAT()** 用于连接两个字段。对于字段中的空格，使用 **TRIM()** 可以去除首尾空格。

```sql
SELECT CONCAT (col1, col2) AS c FROM mytable;
```

结果为

![image-20210805213238161](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210805213238.png)

# 12、函数

主要说明下 MySQL 的聚合函数

## 汇总

|  函 数  |      说 明       |
| :-----: | :--------------: |
|  AVG()  | 返回某列的平均值 |
| COUNT() |  返回某列的行数  |
|  MAX()  | 返回某列的最大值 |
|  MIN()  | 返回某列的最小值 |
|  SUM()  |  返回某列值之和  |

AVG() 会忽略 NULL 行。

使用 DISTINCT 可以汇总不同的值。

```sql
SELECT AVG(DISTINCT col1) AS avgResult FROM mytable;
```

##  文本处理

|   函数    |      说明      |
| :-------: | :------------: |
|  LEFT()   |   左边的字符   |
|  RIGHT()  |   右边的字符   |
|  LOWER()  | 转换为小写字符 |
|  UPPER()  | 转换为大写字符 |
|  LTRIM()  | 去除左边的空格 |
|  RTRIM()  | 去除右边的空格 |
| LENGTH()  |      长度      |
| SOUNDEX() |  转换为语音值  |

**SOUNDEX()** 可以将一个字符串转换为描述其语音表示的字母数字模式。

##  日期和时间处理

- 日期格式：YYYY-MM-DD
- 时间格式：HH:<zero-width space>MM:SS

|     函 数     |             说 明              |
| :-----------: | :----------------------------: |
|   ADDDATE()   |    增加一个日期（天、周等）    |
|   ADDTIME()   |    增加一个时间（时、分等）    |
|   CURDATE()   |          返回当前日期          |
|   CURTIME()   |          返回当前时间          |
|    DATE()     |     返回日期时间的日期部分     |
|  DATEDIFF()   |        计算两个日期之差        |
|  DATE_ADD()   |     高度灵活的日期运算函数     |
| DATE_FORMAT() |  返回一个格式化的日期或时间串  |
|     DAY()     |     返回一个日期的天数部分     |
|  DAYOFWEEK()  | 对于一个日期，返回对应的星期几 |
|    HOUR()     |     返回一个时间的小时部分     |
|   MINUTE()    |     返回一个时间的分钟部分     |
|    MONTH()    |     返回一个日期的月份部分     |
|     NOW()     |       返回当前日期和时间       |
|   SECOND()    |      返回一个时间的秒部分      |
|    TIME()     |   返回一个日期时间的时间部分   |
|    YEAR()     |     返回一个日期的年份部分     |

interval 关键字，可和日期使用，同时还可和 DATE_SUB(),SUBDATE(),ADDDATE() 等函数使用。DATE_SUB(a, b) 和 SUBDATE(a, b) 相同，返回的都是比 a 日期少 b 时间的日期。ADDDATE(a, b) 则是返回比 a 日期多 b 的时间的日期。

```sql
SELECT NOW() - INTERVAL 2 HOUR;
```

上述返回的是比现在时间少两小时的日期。

```sql
SELECT NOW(), SUBDATE(NOW(), INTERVAL 2 DAY);
```

上述返回的是当前日期和比现在日期少两天的日期。

## 数值处理

|  函数  |  说明  |
| :----: | :----: |
| SIN()  |  正弦  |
| COS()  |  余弦  |
| TAN()  |  正切  |
| ABS()  | 绝对值 |
| SQRT() | 平方根 |
| MOD()  |  余数  |
| EXP()  |  指数  |
|  PI()  | 圆周率 |
| RAND() | 随机数 |

## FROM_UNIXTIME

即将时间戳转换为日期类型进行显示。可将 Java 中 System.currentTimeMillis() 产生的时间戳转换为日期

用法：FROM_UNIXTIME(unix_timestamp,format)，不过要注意的是，应当把 unix_timestamp 转换为秒为单位，也就是说 System.currentTimeMillis() 产生的时间戳应该 /100

format 可选参数如下：

```html
%M 月名字(January……December)
%W 星期名字(Sunday……Saturday)
%D 有英语前缀的月份的日期(1st, 2nd, 3rd, 等等。）
%Y 年, 数字, 4 位
%y 年, 数字, 2 位
%a 缩写的星期名字(Sun……Sat)
%d 月份中的天数, 数字(00……31)
%e 月份中的天数, 数字(0……31)
%m 月, 数字(01……12)
%c 月, 数字(1……12)
%b 缩写的月份名字(Jan……Dec)
%j 一年中的天数(001……366)
%H 小时(00……23)
%k 小时(0……23)
%h 小时(01……12)
%I 小时(01……12)
%l 小时(1……12)
%i 分钟, 数字(00……59)
%r 时间,12 小时(hh:mm:ss [AP]M)
%T 时间,24 小时(hh:mm:ss)
%S 秒(00……59)
%s 秒(00……59)
%p AM或PM
%w 一个星期中的天数(0=Sunday ……6=Saturday ）
%U 星期(0……52), 这里星期天是星期的第一天
%u 星期(0……52), 这里星期一是星期的第一天
%% 一个文字“%”。
```

# 13、分组

## GROUP BY

把具有相同的数据值的行放在同一组中。可对选择的数据使用函数进行处理，后面也可使用 ORDER BY 进行排序，WHERE 过滤行。

```sql
SELECT col1, AVG(col1) FROM mytable WHERE col1 >= 2 GROUP BY col1 ORDER BY col1 DESC;
```

分组规定：

- GROUP BY 子句出现在 WHERE 子句之后，ORDER BY 子句之前；
- 除了汇总字段外，SELECT 语句中的每一字段都必须在 GROUP BY 子句中给出；
- NULL 的行会单独分为一组；
- 大多数 SQL 实现不支持 GROUP BY 列具有可变长度的数据类型。

## HAVING

HAVING 子句对 GROUP BY 子句设置条件的方式与 WHERE 和 SELECT 的交互方式类似。WHERE 搜索条件在进行分组操作之前应用；而 HAVING 搜索条件在进行分组操作之后应用。HAVING 语法与 WHERE 语法类似，但 HAVING 可以包含聚合函数。

```sql
SELECT class FROM courses GROUP BY class HAVING COUNT(student) >= 5; 
```

# 14、子查询

子查询中只能返回一个字段的数据。可以将子查询的结果作为 WHRER 语句的过滤条件：

```sql
SELECT * FROM mytable WHERE col1 IN (SELECT col1 FROM mytable2);
```

```sql
SELECT col2, (SELECT col1 FROM mytable2 WHERE mytable.`col1` = mytable2.`col1`) AS others FROM mytable; 
```

以上代码会将 mytable 中所有的 col2 选出，当 mytable 中 col2 对应 col1 和 mytable2 中对应的 col1 相等时，输出到 col2 对应的 others 中，否则输出 null

![image-20210806155056174](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210806155057.png)

# 15、连接

连接用于连接多个表，使用 JOIN 关键字，并且条件语句使用 ON 而不是 WHERE。

连接可以替换子查询，并且比子查询的效率一般会更快。

在连接过程中可使用 AS 对列名、表明等取别名，简化操作。

## 内连接

内连接表示交集，又称等值连接，使用 INNER JOIN 关键字。

```sql
SELECT stu.`Name`, gra.`Grade` FROM student AS stu 
INNER JOIN grade AS gra 
ON stu.`stuNO` = gra.`stuNo`; 
```

也可使用 WHERE 进行等值匹配

```sql
SELECT stu.`Name`, gra.`Grade` FROM student AS stu, grade AS gra 
WHERE stu.`stuNO` = gra.`stuNo`;
```

## 自连接

自连接可以看成内连接的一种，只是连接的表是自身而已。

子查询：

```sql
SELECT stuNo, `Name` FROM student 
WHERE Class IN 
(SELECT Class FROM student WHERE NAME = "xx");
```

自连接：

```sql
SELECT stu1.stuNo, stu1.Name FROM student AS stu1 
INNER JOIN student AS stu2 
ON stu1.`Class` = stu2.`Class` AND stu2.`Name` = "xx";
```

## 自然连接

内连接和自然连接的区别：内连接提供连接的列，而自然连接自动连接**所有同名列**。

```sql
SELECT student.name, grade.`Grade` FROM student NATURAL JOIN grade;
```

## 外连接

外连接保留了没有关联的那些行，分为左外连接，右外连接以及全外连接。**MySQL 中不支持全外连接**。

**左外连接**的结果集包括 LEFT OUTER子句中指定的左表的所有行，而不仅仅是联接列所匹配的行。如果左表的某行在右表中没有匹配行，则在相关联的结果集行中右表的所有选择列表列均为空值。 

**右外连接**是左向外联接的反向联接。将返回右表的所有行。如果右表的某行在左表中没有匹配行，则将为左表返回空值。    

**全外连接**返回左表和右表中的所有行。当某行在另一个表中没有匹配行时，则另一个表的选择列表列包含空值。如果表之间有匹配行，则整个结果集行包含基表的数据值。  

```sql
SELECT student.`Name`, student.`Class`, grade.`grade` 
FROM student LEFT JOIN grade 
ON student.`stuNO` = grade.`stuNO`;
```

```sql
SELECT student.`Name`, student.`Class`, grade.`grade` 
FROM student RIGHT JOIN grade 
ON student.`stuNO` = grade.`stuNO`;
```

# 16、组合查询

使用 **UNION** 来组合两个查询，如果第一个查询返回 M 行，第二个查询返回 N 行，那么组合查询的结果一般为 M+N 行。默认会去除相同行，如果需要保留相同行，使用 UNION ALL。

每个查询必须包含相同的列数、表达式和聚集函数。只能包含一个 ORDER BY 子句，并且必须位于语句的最后。

```sql
SELECT student.`stuNO` FROM student WHERE student.`Name` = "xx" UNION SELECT grade.`stuNO` FROM grade ORDER BY stuNO;
```

# 17、视图

在 SQL 中，视图是基于 SQL 语句的结果集的可视化的表。视图是虚拟的表，本身不包含数据，也就不能对其进行索引操作，对视图的操作和对普通表的操作一样。

视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。我们可以向视图添加 SQL 函数、WHERE 以及 JOIN 语句，我们也可以提交数据，就像这些来自于某个单一的表。

视图的优点：

- 简化复杂的 SQL 操作，比如复杂的连接；
- 只使用实际表的一部分数据；
- 通过只给用户访问视图的权限，保证数据的安全性；
- 更改数据格式和表示。

语法：

```sql
CREATE VIEW view_name AS
SELECT column1, column2, ... FROM table_name 
WHERE condition;
```

```sql
CREATE VIEW myView AS SELECT student.`stuNO` FROM student WHERE class = xxx;
```

# 18、存储过程

存储过程可以看成是对一系列 SQL 操作的批处理。

使用存储过程的优点：

- 代码封装，保证了一定的安全性；
- 代码复用；
- 由于是预先编译，因此具有很高的性能。

使用 call 来调用 存储过程

# 19、游标

在存储过程中使用游标可以对一个结果集进行移动遍历。

游标主要用于交互式应用，其中用户需要对数据集中的任意行进行浏览和修改。

使用游标的四个步骤：

1. 声明游标，这个过程没有实际检索出数据；
2. 打开游标；
3. 取出数据；
4. 关闭游标；

# 20、触发器

触发器（trigger）是SQL server 提供给程序员和数据分析员来保证数据完整性的一种方法，它是与表事件相关的特殊的存储过程，它的执行不是由程序调用，也不是手工启动，而是由事件来触发，当对一个表进行操作（ insert，delete， update）时就会激活它执行。

触发器必须指定在语句执行之前还是之后自动执行，之前执行使用 BEFORE 关键字，之后执行使用 AFTER 关键字。BEFORE 用于数据验证和净化，AFTER 用于审计跟踪，将修改记录到另外一张表中。

触发器的优点：

1. 触发器是自动的。当对表中的数据做了任何修改之后立即被激活。
2. 触发器可以通过数据库中的相关表进行层叠修改。
3. 触发器可以强制限制。这些限制比用CHECK约束所定义的更复杂。与CHECK约束不同的是，触发器可以引用其他表中的列。

分类：

1. **DML(数据操作语言,Data Manipulation Language)触发器**
2. **DDL(数据定义语言,Data Definition Language)触发器**
3. **登录触发器**

INSERT 触发器包含一个名为 NEW 的虚拟表。

DELETE 触发器包含一个名为 OLD 的虚拟表，并且是只读的。

UPDATE 触发器包含一个名为 NEW 和一个名为 OLD 的虚拟表，其中 NEW 是可以被修改的，而 OLD 是只读的。

MySQL 不允许在触发器中使用 CALL 语句，也就是不能调用存储过程

# 21、事务管理

- 事务（transaction）指一组 SQL 语句；
- 回退（rollback）指撤销指定 SQL 语句的过程；
- 提交（commit）指将未存储的 SQL 语句结果写入数据库表；
- 保留点（savepoint）指事务处理中设置的临时占位符（placeholder），你可以对它发布回退（与回退整个事务处理不同）。

不能回退 SELECT、CREATE 和 DROP 语句

MySQL 的事务提交默认是隐式提交，每执行一条语句就把这条语句当成一个事务然后进行提交。当出现 START TRANSACTION 语句时，会关闭隐式提交；当 COMMIT 或 ROLLBACK 语句执行后，事务会重新恢复隐式提交。

事务特性：

- 原子性：事务必须是一个自动工作的单元，要么全部执行，要么全部不执行。
- 一致性：事务结束的时候，所有的内部数据都是正确的。
- 隔离性：并发多个事务时，各个事务不干涉内部数据，处理的都是另外一个事务处理之前或之后的数据。
- 持久性：事务提交之后，数据是永久性的，不可再回滚。

常用语句：

- Begin Transaction：标记事务开始。
- Commit Transaction：事务已经成功执行，数据已经处理妥当。
- Rollback Transaction：数据处理过程中出错，回滚到没有处理之前的数据状态，或回滚到事务内部的保存点。
- Save Transaction：事务内部设置的保存点，就是事务可以不全部回滚，只回滚到这里，保证事务内部不出错的前提下。

设置 autocommit 为 0 可以取消自动提交；autocommit 标记是针对每个连接而不是针对服务器的。

如果没有设置保留点，ROLLBACK 会回退到 START TRANSACTION 语句处；如果设置了保留点，并且在 ROLLBACK 中指定该保留点，则会回退到该保留点。

事务间的影响：脏读、不可重复读、幻读、丢失更新

## 事务实现的原理

- 日志文件(redo log 和 undo log)
- 锁技术
- MVCC

## redo log 与 undo log

### redo log

redo log 叫做重做日志，是用来实现事务的持久性。该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log）,前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都会存到该日志中。

原始数据库中表数据为

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210806185819.webp)

```csharp
start transaction;
select balance from bank where name="zhangsan";
// 生成 重做日志 balance=600
update bank set balance = balance - 400; 
// 生成 重做日志 amount=400
update finance set amount = amount + 400;
```

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210806185902.webp)

### redo log 作用

mysql 为了提升性能不会把每次的修改都实时同步到磁盘，而是会先存到 Beffer Pool(缓冲池)里头，把这个当作缓存来用。然后使用后台线程去做缓冲池和磁盘之间的同步。

但是万一由于停电等原因导致丢部分已提交事务的修改信息，因此引入了redo log来记录已成功提交事务的修改信息，并且会把redo log持久化到磁盘，系统重启之后在读取redo log恢复最新数据。

redo log是用来恢复数据的，用于保障，已提交事务的持久化特性（记录了已经提交的操作）

### undo log

undo log 叫做回滚日志，用于记录数据被修改前的信息。undo log主要记录的是数据的逻辑变化，为了在发生错误时回滚之前的操作，需要将之前的操作都记录下来，然后在发生错误时才可以回滚。

同样以上述例子为例，有

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210806190207.webp)

每次写入数据或者修改数据之前都会把修改前的信息记录到 undo log。

### undo log 作用

undo log 记录事务修改之前版本的数据信息，因此假如由于系统错误或者rollback操作而回滚的话可以根据undo log的信息来进行回滚到没被修改前的状态。

undo log是用来回滚数据的用于保障，未提交事务的原子性

## mysql 锁技术

当多个请求里有读请求，又有修改请求时必须有一种措施来进行并发控制。不然很有可能会造成不一致。只需用两种锁的组合来对读写请求进行控制即可，这两种锁被称为：

- 共享锁(shared lock)

  又叫做"读锁"读锁是可以共享的，或者说多个读请求可以共享一把锁读数据，不会造成阻塞。

- 排他锁(exclusive lock)

  又叫做"写锁"写锁会排斥其他所有获取锁的请求，一直阻塞，直到写入完成释放锁

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210806190550.webp)

通过读写锁，可以做到读读可以并行，但是不能做到写读，写写并行

## MVCC

MVCC (MultiVersion Concurrency Control) 叫做多版本并发控制。一般情况下，事务性储存引擎不是只使用表锁，行加锁的处理数据，而是结合了MVCC机制，以处理更多的并发问题。MVCC 处理高并发能力最强，但系统开销比最大（较表锁、行级锁），这是最求高并发付出的代价。

MVCC 的主要实现思想是通过数据多版本来做到读写分离。从而实现不加锁读进而做到读写并行。MVCC 在 mysql 中的实现依赖的是 undo log 与 read view 。

- undo log :undo log 中记录某行数据的多个版本的数据。
- read view :用来判断当前版本数据的可见性

不同存储引擎的MVCC实现是不同的，典型的有乐观（optimistic）并发控制和悲观（pessimistic）并发控制。

# 22、字符集

- 字符集：字母和符号的集合；
- 编码：某个字符集成员的内部表示；
- 校对字符：如何比较，主要用于排序和分组。

# 23、权限管理

MySQL 的账户信息保存在 mysql 这个数据库中。

## 创建用户

新用户没有任何权限

```sql
CREATE USER myuser IDENTIFIED BY 'mypassword';
```

## 修改账户名

```sql
RENAME USER myuser TO newuser;
```

## 删除账户

```sql
DROP USER myuser;
```

## 查看权限

```sql
SHOW GRANTS FOR myuser;
```

## 授予权限

格式：若想授予所有权限，则使用 ALL

```sql
GRANT privileges ON databasename.tablename TO 'username'@'host'
```

```sql
GRANT ALL ON db_studentinfo.* TO 'myuser'@'%';
```

## 删除权限

GRANT 和 REVOKE 可在几个层次上控制访问权限：

- 整个服务器，使用 GRANT ALL 和 REVOKE ALL；
- 整个数据库，使用 ON database.*；
- 特定的表，使用 ON database.table；
- 特定的列；
- 特定的存储过程。

```sql
REVOKE SELECT, INSERT ON db_studentinfo.* FROM 'myuser'@'%';
```

## 更改密码

```sql
SET PASSWORD FOR myuser = PASSWORD('newpassword');
```

MySQL 密码的加密方式：

- password() 函数，使用MySQLSHA1（安全Hash算法）进行加密
- old_password() 函数
- encode 和 decode 的加密
- MD5 函数
