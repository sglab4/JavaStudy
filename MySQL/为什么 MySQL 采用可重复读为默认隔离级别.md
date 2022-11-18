参考 https://www.cnblogs.com/shoshana-kong/p/10516404.html

在 MySQL 中，复制采用的是主从复制，基于 bin log 实现，bin log 有如下三种格式：

- **statement:记录的是修改SQL语句**
- **row：记录的是每行实际数据的变更**
- **mixed：statement和row模式的混合**

**在 MySQL 5.0 之前，binlog只支持`STATEMENT`这种格式！而这种格式在读已提交(Read Commited)这个隔离级别下主从复制是有bug的，因此Mysql将可重复读(Repeatable Read)作为默认的隔离级别**

接下来说明在 statement 格式下，读已提交的 bug

在主（master）上执行下列语句

```sql
select * from test；
```

输出如下

```text
+---+
| b |
+---+
| 3 |
+---+
1 row in set
```

在从（slave）执行上述语句，得到

```text
Empty set
```

那么可发现出现了数据一致性问题，原因就是在 master 上执行的顺序为先删后插，而此时 binlog 为 STATEMENT 格式，它记录的顺序为先插后删，slave 同步的是 binlog，因此出现数据不一致

解决方法：

1. 隔离级别设为**可重复读(Repeatable Read)**,在该隔离级别下引入间隙锁。当`Session 1`执行delete语句时，会锁住间隙。那么，`Ssession 2`执行插入语句就会阻塞住
2. 将binglog的格式修改为row格式，此时是基于行的复制，自然就不会出现sql执行顺序不一样的问题，但这个格式在mysql5.1版本开始才引入。因此由于历史原因，mysql将默认的隔离级别设为**可重复读(Repeatable Read)**，保证主从复制不出问题

在互联网项目中使用读已提交作为默认隔离级别的原因：

- **读未提交**会出现一个事务读到另一个事务未提交的数据，出现脏读。
- **串行化中**每个次读操作都会加锁，快照读失效，一般是使用mysql自带分布式事务功能时才使用该隔离级别

因此剩余**可重复读（Repeatable Read，RR）**和**读已提交（Read Commited，RC）**

原因：

1. **在RR隔离级别下，存在间隙锁，导致出现死锁的几率比RC大的多**
2. **在RR隔离级别下，条件列未命中索引会锁表！而在RC隔离级别下，只锁行**

