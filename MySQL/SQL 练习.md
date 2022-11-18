# 1、595 Big Countries

题目描述：查找面积超过 3,000,000 或者人口数超过 25,000,000 的国家。

```html
+-----------------+------------+------------+--------------+---------------+
| name            | continent  | area       | population   | gdp           |
+-----------------+------------+------------+--------------+---------------+
| Afghanistan     | Asia       | 652230     | 25500100     | 20343000      |
| Albania         | Europe     | 28748      | 2831741      | 12960000      |
| Algeria         | Africa     | 2381741    | 37100000     | 188681000     |
| Andorra         | Europe     | 468        | 78115        | 3712000       |
| Angola          | Africa     | 1246700    | 20609294     | 100990000     |
+-----------------+------------+------------+--------------+---------------+
```

```sql
DROP TABLE
IF
    EXISTS World;
CREATE TABLE World ( NAME VARCHAR ( 255 ), continent VARCHAR ( 255 ), AREA INT, population INT, gdp INT );
INSERT INTO World ( NAME, continent, AREA, population, gdp )
VALUES
    ( 'Afghanistan', 'Asia', '652230', '25500100', '203430000' ),
    ( 'Albania', 'Europe', '28748', '2831741', '129600000' ),
    ( 'Algeria', 'Africa', '2381741', '37100000', '1886810000' ),
    ( 'Andorra', 'Europe', '468', '78115', '37120000' ),
    ( 'Angola', 'Africa', '1246700', '20609294', '1009900000' );
```

解题思路：使用 WHERE 和 OR 直接选择即可

```sql
SELECT `name`, population, `area` FROM World 
WHERE `area` > 3000000 OR population > 25000000;
```

# 2、627 Swap Salary

题目描述：只用一个 SQL 查询，将 sex 字段反转。

```html
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
```

```html
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
```

```sql
DROP TABLE
IF
    EXISTS salary;
CREATE TABLE salary ( id INT, NAME VARCHAR ( 100 ), sex CHAR ( 1 ), salary INT );
INSERT INTO salary ( id, NAME, sex, salary )
VALUES
    ( '1', 'A', 'm', '2500' ),
    ( '2', 'B', 'f', '1500' ),
    ( '3', 'C', 'm', '5500' ),
    ( '4', 'D', 'f', '500' );
```

解题思路：由于 sex 字段仅使用 'm' 和 'f'，因此字段翻转，用到异或 ^ 运算符。

```text
'f' ^ ('m' ^ 'f') = 'm' ^ ('f' ^ 'f') = 'm'
'm' ^ ('m' ^ 'f') = 'f' ^ ('m' ^ 'm') = 'f'
```

注意，需要将 sex 字段中的值转换为 ASCII 后进行异或运算，再转换为 CHAR 类型。

```sql
UPDATE salary SET sex = CHAR(ASCII(sex) ^ ASCII('m') ^ ASCII('f'));
```

# 3、620 Not Boring Movies

题目描述：查找 id 为奇数，并且description 不是 boring 的电影，按 rating 降序。

```html
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
```

```html
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
```

```sql
DROP TABLE
IF
    EXISTS cinema;
CREATE TABLE cinema ( id INT, movie VARCHAR ( 255 ), description VARCHAR ( 255 ), rating FLOAT ( 2, 1 ) );
INSERT INTO cinema ( id, movie, description, rating )
VALUES
    ( 1, 'War', 'great 3D', 8.9 ),
    ( 2, 'Science', 'fiction', 8.5 ),
    ( 3, 'irish', 'boring', 6.2 ),
    ( 4, 'Ice song', 'Fantacy', 8.6 ),
    ( 5, 'House card', 'Interesting', 9.1 );
```

解题思路：取余运算可使用运算符 % 或者 MOD 函数，例如 MOD(4, 2)

```sql
SELECT * FROM cinema 
WHERE id % 2 = 1 AND description != 'boring' 
ORDER BY rating DESC;

SELECT * FROM cinema 
WHERE MOD(id, 2) = 1 AND description != 'boring' 
ORDER BY rating DESC;
```

# 4、596 Classes More Than 5 Students

题目描述：查找有五名及以上 student 的 class。

```html
+---------+------------+
| student | class      |
+---------+------------+
| A       | Math       |
| B       | English    |
| C       | Math       |
| D       | Biology    |
| E       | Math       |
| F       | Computer   |
| G       | Math       |
| H       | Math       |
| I       | Math       |
+---------+------------+
```

```html
+---------+
| class   |
+---------+
| Math    |
+---------+
```

```sql
DROP TABLE
IF
    EXISTS courses;
CREATE TABLE courses ( student VARCHAR ( 255 ), class VARCHAR ( 255 ) );
INSERT INTO courses ( student, class )
VALUES
    ( 'A', 'Math' ),
    ( 'B', 'English' ),
    ( 'C', 'Math' ),
    ( 'D', 'Biology' ),
    ( 'E', 'Math' ),
    ( 'F', 'Computer' ),
    ( 'G', 'Math' ),
    ( 'H', 'Math' ),
    ( 'I', 'Math' );
```

解题思路：本题用到了 GROUP BY，将相同的 class 分为一组，然后查找不同分组中总人数大于 5 的 class，使用到了 HAVING，可联合聚合函数，用于对分组后进行过滤。

```sql
SELECT class FROM courses 
GROUP BY class 
HAVING COUNT(student) >= 5; 
```

# 5、182 Duplicate Emails

题目描述：要求查找重复地址。

邮件地址表：

```html
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
```

重复地址：

```html
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```

```sql
DROP TABLE
IF
    EXISTS Person;
CREATE TABLE Person ( Id INT, Email VARCHAR ( 255 ) );
INSERT INTO Person ( Id, Email )
VALUES
    ( 1, 'a@b.com' ),
    ( 2, 'c@d.com' ),
    ( 3, 'a@b.com' );
```

解题思路：本题和 596 相似，都用到了 GROUP BY ... HAVING，按照 Email 进行分组，若同一组中 Id 大于 1，则输出  Email 即可。

```sql
SELECT Email FROM person 
GROUP BY Email 
HAVING COUNT(Id) > 1;
```

# 6、196 Delete Duplicate Emails

题目描述：删除重复的邮件地址，保留重复邮件 Id 较小的那个。

邮件地址表：

```html
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
```

删除重复地址：

```html
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
```

```sql
DROP TABLE
IF
    EXISTS Person;
CREATE TABLE Person ( Id INT, Email VARCHAR ( 255 ) );
INSERT INTO Person ( Id, Email )
VALUES
    ( 1, 'a@b.com' ),
    ( 2, 'c@d.com' ),
    ( 3, 'a@b.com' );
```

解题思路：本题有两种解法：

1. 自连接删除，注意 WHERE 中的条件，删除的是 Id 中除最小值外的较大值

   ```sql
   DELETE p1 FROM person AS p1, person AS p2 
   WHERE p1.`Id` > p2.`Id` AND p1.`Email` = p2.`Email`;
   ```

   ```sql
   DELETE p1 FROM 
   person AS p1 INNER JOIN person AS p2
   ON p1.`Email` = p2.`Email` AND p1.`Id` > p2.`Id`;
   ```

2. 子查询，注意额外嵌套了一个 SELECT 语句，而且必须加 AS 起别名，不加 SELECT 的话将会发生 "You can't specify target table 'Person' for update in FROM clause" 错误。

   ```sql
   DELETE FROM person 
   WHERE id NOT IN 
   (SELECT id FROM 
    (SELECT MIN(id) AS id FROM person GROUP BY Email) 
    AS m);
   ```

# 7、175 Combine Two Tables

题目描述：查找 FirstName, LastName, City, State 数据，而不管一个用户有没有填地址信息。

Person 表：

```html
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId is the primary key column for this table.
```

Address 表：

```html
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId is the primary key column for this table.
```

```sql
DROP TABLE
IF
    EXISTS Person;
CREATE TABLE Person ( PersonId INT, FirstName VARCHAR ( 255 ), LastName VARCHAR ( 255 ) );
DROP TABLE
IF
    EXISTS Address;
CREATE TABLE Address ( AddressId INT, PersonId INT, City VARCHAR ( 255 ), State VARCHAR ( 255 ) );
INSERT INTO Person ( PersonId, LastName, FirstName )
VALUES
    ( 1, 'Wang', 'Allen' );
INSERT INTO Address ( AddressId, PersonId, City, State )
VALUES
    ( 1, 2, 'New York City', 'New York' );
```

解题思路：两个表的联系是 personId 相同的值，因此将两个表 personId 相同的值的内容输出，当 address 表中没有该 personId 时，输出空值，因此可发现需要用到左连接。

```sql
SELECT FirstName, LastName, City, State 
FROM person LEFT JOIN address 
ON person.`PersonId` = address.`PersonId`;
```

# 8、181 Employees Earning More Than Their Managers

题目描述：查找薪资大于其经理薪资的员工信息。

Employee 表：

```html
+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+
```

```sql
DROP TABLE
IF
    EXISTS Employee;
CREATE TABLE Employee ( Id INT, NAME VARCHAR ( 255 ), Salary INT, ManagerId INT );
INSERT INTO Employee ( Id, NAME, Salary, ManagerId )
VALUES
    ( 1, 'Joe', 70000, 3 ),
    ( 2, 'Henry', 80000, 4 ),
    ( 3, 'Sam', 60000, NULL ),
    ( 4, 'Max', 90000, NULL );
```

解题思路：很明显，本题需要使用自连接将两个 Employee 表连接起来进行查询，注意条件为两个，一个是员工 Id 和自己经理 Id 相同，另一个是员工薪资大于经理薪资。

```sql
SELECT e1.`NAME` FROM Employee AS e1 INNER JOIN Employee AS e2 
ON e1.`ManagerId` = e2.`Id` AND e1.`Salary` > e2.`Salary`;
```

# 9、183 Customers Who Never Order

题目描述：查找没有订单的顾客信息。

Customers 表：

```html
+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
```

Orders 表：

```html
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
```

```html
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```

```sql
DROP TABLE
IF
    EXISTS Customers;
CREATE TABLE Customers ( Id INT, NAME VARCHAR ( 255 ) );
DROP TABLE
IF
    EXISTS Orders;
CREATE TABLE Orders ( Id INT, CustomerId INT );
INSERT INTO Customers ( Id, NAME )
VALUES
    ( 1, 'Joe' ),
    ( 2, 'Henry' ),
    ( 3, 'Sam' ),
    ( 4, 'Max' );
INSERT INTO Orders ( Id, CustomerId )
VALUES
    ( 1, 3 ),
    ( 2, 1 );
```

解题思路：

1. 左链接

   将 Customers 和 Orders 采用左连接连接起来，ON 的条件是 Customers.`Id` = orders.`CustomerId` ，然后查询 Customers 表中的 name，这时所有的 name 将会被查出，结果如下

   ![image-20210811172743444](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210811172750.png)

   这时，再使用 WHERE 去除 CustomerId 为 null 的值即可。

   ```sql
   SELECT `name`, CustomerId
   FROM
       Customers
       LEFT JOIN orders
       ON Customers.`Id` = orders.`CustomerId`
   WHERE orders.`CustomerId` IS NULL;
   ```

2. 子查询

   子查询很好理解，直接从 Customers 中查询不在 CustomerId 中的 name 即可。

   ```sql
   SELECT 
       `name` 
   FROM 
       Customers
   WHERE    
       Customers.`Id` NOT IN 
       (SELECT 
           CustomerId 
        FROM 
            orders);
   ```

# 10、184 Department Highest Salary

题目描述：查找不同部门中收入最高者的信息

Employee 表：

```html
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
+----+-------+--------+--------------+
```

Department 表：

```html
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
```

```html
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+
```

```sql
DROP TABLE IF EXISTS Employee;
CREATE TABLE Employee ( Id INT, NAME VARCHAR ( 255 ), Salary INT, DepartmentId INT );
DROP TABLE IF EXISTS Department;
CREATE TABLE Department ( Id INT, NAME VARCHAR ( 255 ) );
INSERT INTO Employee ( Id, NAME, Salary, DepartmentId )
VALUES
    ( 1, 'Joe', 70000, 1 ),
    ( 2, 'Henry', 80000, 2 ),
    ( 3, 'Sam', 60000, 2 ),
    ( 4, 'Max', 90000, 1 );
INSERT INTO Department ( Id, NAME )
VALUES
    ( 1, 'IT' ),
    ( 2, 'Sales' );
```

解题思路：要求查找出的内容为部门名称，员工姓名，最高工资，因此首先需要在 Employee 表中查找出每个部门员工的最高工资和部门 Id，然后选出 Employee 中的部门 Id、薪资大小均相等的内容。

```sql
SELECT E.`DepartmentId`, E.`NAME`, D.`NAME`
FROM
    Employee AS E,
    Department AS D,
    (SELECT DepartmentId, MAX(Salary) AS Salary
     FROM Employee
     GROUP BY DepartmentId
     ) AS M
WHERE 
    E.`DepartmentId` = D.`Id`
    AND E.`DepartmentId` = M.`DepartmentId`
    AND E.`Salary` = M.`Salary`;
```

# 11、176 Second Highest Salary

题目描述：查找薪资第二高的员工的薪资，若未找到则返回 null

```html
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

```html
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

```sql
DROP TABLE
IF
    EXISTS Employee;
CREATE TABLE Employee ( Id INT, Salary INT );
INSERT INTO Employee ( Id, Salary )
VALUES
    ( 1, 100 ),
    ( 2, 200 ),
    ( 3, 300 );
```

解题思路：本题需要用到 limit，按工资对员工进行分组，然后降序排序，使用 limit 返回第二个值即可。

```sql
select ifnull ((select distinct salary from Employee order by salary desc limit 1, 1), null) as SecondHighestSalary;
```

# 12、177 Nth Highest Salary

题目描述：查找工资第 N 高的员工。

```sql
DROP TABLE
IF
    EXISTS Employee;
CREATE TABLE Employee ( Id INT, Salary INT );
INSERT INTO Employee ( Id, Salary )
VALUES
    ( 1, 100 ),
    ( 2, 200 ),
    ( 3, 300 );
```

解题思路：和 176 相同，先按 Salary 对员工进行排序，然后使用 limit 返回第 N 高的员工，limit N-1, 1;

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  Set N = N - 1;
  RETURN (
      # Write your MySQL query statement below.
      select ifnull((select distinct Salary from Employee order by Salary desc limit N, 1), null)
      #select ifnull((select distinct Salary from Employee order by Salary desc limit N,1),null) as getNthHighestSalary
  );
END
```

# 13、178 Rank Scores

题目描述：对不同的得分进行排序，并统计排名。

```html
+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
```

```html
+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```

```sql
DROP TABLE
IF
    EXISTS Scores;
CREATE TABLE Scores ( Id INT, Score DECIMAL ( 3, 2 ) );
INSERT INTO Scores ( Id, Score )
VALUES
    ( 1, 4.1 ),
    ( 2, 4.1 ),
    ( 3, 4.2 ),
    ( 4, 4.2 ),
    ( 5, 4.3 ),
    ( 6, 4.3 );
```

解题思路：

以以下表为例说明思路

|  Id  | score | 大于等于该 score 的 score 数量 | 排名 |
| :--: | :---: | :----------------------------: | :--: |
|  1   |  4.1  |               3                |  3   |
|  2   |  4.2  |               2                |  2   |
|  3   |  4.3  |               1                |  1   |

对该表自连接，选出大于等于每个 score 的记录：

```sql
SELECT
	*
FROM
    Scores S1
    INNER JOIN Scores S2
    ON S1.score <= S2.score
ORDER BY
    S1.score DESC, S1.Id;
```

那么，可得到下表

| S1.Id | S1.score | S2.Id | S2.score |
| :---: | :------: | :---: | :------: |
|   3   |   4.3    |   3   |   4.3    |
|   2   |   4.2    |   2   |   4.2    |
|   2   |   4.2    |   3   |   4.3    |
|   1   |   4.1    |   1   |   4.1    |
|   1   |   4.1    |   2   |   4.2    |
|   1   |   4.1    |   3   |   4.3    |

根据上表，统计相同的 s1.score 的数量。

```sql
SELECT
    S1.score 'Score',
    COUNT(*) 'Rank'
FROM
    Scores S1
    INNER JOIN Scores S2
    ON S1.score <= S2.score
GROUP BY
    S1.id, S1.score
ORDER BY
    S1.score DESC, S1.Id;
```

得到表如下：

| score | Rank |
| :---: | :--: |
|  4.3  |  1   |
|  4.2  |  2   |
|  4.1  |  3   |

但是，以上解法有一点问题，当出现相同的分数时，如

|  Id  | score |
| :--: | :---: |
|  1   |  4.1  |
|  2   |  4.2  |
|  3   |  4.2  |

得到的结果为

| score | Rank |
| :---: | :--: |
|  4.2  |  2   |
|  4.2  |  2   |
|  4.1  |  3   |

采用自连接后，查询的表为

| S1.Id | S1.score | S2.Id | S2.score |
| :---: | :------: | :---: | :------: |
|   2   |   4.2    |   3   |   4.2    |
|   2   |   4.2    |   2   |   4.2    |
|   3   |   4.2    |   3   |   4.2    |
|   3   |   4.2    |   2   |   4.2    |
|   1   |   4.1    |   3   |   4.2    |
|   1   |   4.1    |   2   |   4.2    |
|   1   |   4.1    |   1   |   4.1    |

若仍采用 COUNT(*)，则等于 4.2 的内容将会被计算两次，出现错误，因此需要使用 COUNT( DISTINCT S2.score ) 进行去重。注意不要忘记 group by

```sql
SELECT
    S1.score 'Score',
    COUNT( DISTINCT S2.score ) 'Rank'
FROM
    Scores S1
    INNER JOIN Scores S2
    ON S1.score <= S2.score
GROUP BY
    S1.id, S1.score
ORDER BY
    S1.score DESC;
```

# 14、180 Consecutive Numbers

题目描述：查找连续出现三次的数字。

```html
+----+-----+
| Id | Num |
+----+-----+
| 1  |  1  |
| 2  |  1  |
| 3  |  1  |
| 4  |  2  |
| 5  |  1  |
| 6  |  2  |
| 7  |  2  |
+----+-----+
```

```html
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
```

```sql
DROP TABLE
IF
    EXISTS LOGS;
CREATE TABLE LOGS ( Id INT, Num INT );
INSERT INTO LOGS ( Id, Num )
VALUES
    ( 1, 1 ),
    ( 2, 1 ),
    ( 3, 1 ),
    ( 4, 2 ),
    ( 5, 1 ),
    ( 6, 2 ),
    ( 7, 2 );
```

解题思路：因为问的是连续出现三次的 num，因此单个表不能选出，需要使用三个表和 where 进行筛选

```sql
SELECT DISTINCT log1.`Num`
FROM
    `logs` log1,
    `logs` log2`,
    `logs` log3
WHERE
    log1.`Id` + 1 = log2.`Id`
    AND log2.`Id` + 1 = log3.`Id`
    AND log1.`Num` = log2.`Num`
    AND log2.`Num` = log3.`Num`;

```

# 15、626 Exchange Seats

题目描述：给定 seat 表，根据 seat 表选择，要求交换相邻座位的两个学生，如果最后一个座位是奇数，那么不交换这个座位上的学生。

```html
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Abbot   |
|    2    | Doris   |
|    3    | Emerson |
|    4    | Green   |
|    5    | Jeames  |
+---------+---------+
```

```html
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Doris   |
|    2    | Abbot   |
|    3    | Green   |
|    4    | Emerson |
|    5    | Jeames  |
+---------+---------+
```

```sql
DROP TABLE
IF
    EXISTS seat;
CREATE TABLE seat ( id INT, student VARCHAR ( 255 ) );
INSERT INTO seat ( id, student )
VALUES
    ( '1', 'Abbot' ),
    ( '2', 'Doris' ),
    ( '3', 'Emerson' ),
    ( '4', 'Green' ),
    ( '5', 'Jeames' );
```

解题思路：本题 select 分为三个部分，偶数，奇数和最大奇数 id 值，当处理偶数 id 值时，需要对 id 值 - 1，当处理奇数 id 值（不为最大值）时，需要对 id 值 + 1，再处理最大的那个奇数 id 值即可。对于上述 select 得到的结果，使用 union

```sql
-- 处理 id 为偶数
SELECT s1.`id` - 1 `id`, s1.`student`
FROM
    seat s1
WHERE s1.`id` MOD 2 = 0
UNION
-- 处理 id 为奇数，不处理最大的 id 值
SELECT s2.`id` + 1, s2.`student`
FROM
    seat s2
WHERE s2.`id` MOD 2 = 1 AND s2.`id` != (SELECT MAX(s3.`id`) FROM seat s3)
UNION
-- 处理 id 最大的记录
SELECT s4.`id`, s4.`student`
FROM
    seat s4
WHERE s4.`id` MOD 2 = 1 AND s4.`id` = (SELECT MAX(s5.`id`) FROM seat s5)
ORDER BY id;
```















