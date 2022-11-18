参考 https://www.cnblogs.com/jojop/p/13982679.html

# 1、概述

锁是计算机用以协调多个进程间并发访问同一共享资源的一种机制。MySQL中为了保证数据访问的一致性与有效性等功能，实现了锁机制，MySQL中的锁是在服务器层或者存储引擎层实现的。

# 2、行锁和表锁

首先我们来了解行锁与表锁的基本概念，从名字中我们就可以了解：表锁就是对整张表进行加锁，而行锁则是锁定某行、某几行数据或者行之间的间隙。

各引擎对锁的支持情况如下：

|        | 行锁 | 表锁 | 页锁 |
| ------ | ---- | ---- | ---- |
| MyISAM |      | √    |      |
| BDB    |      | √    | √    |
| InnoDB | √    | √    |      |

## 行锁

行锁是作用在索引的基础上的，即使在创建表时没有定义索引，InnoDB也会创建一个聚簇索引并将其作为锁作用的索引。

每一个InnoDB表都需要一个聚簇索引，有且只有一个。如果你为该表表定义一个主键，那么MySQL将使用主键作为聚簇索引；如果你不为定义一个主键，那么MySQL将会把第一个唯一索引（而且要求NOT NULL）作为聚簇索引；如果上诉两种情况都不存在，那么MySQL将自动创建一个名字为`GEN_CLUST_INDEX`的隐藏聚簇索引。

因为是聚簇索引，所以B+树上的叶子节点都存储了数据行，那么如果现在是二级索引呢？InnoDB中的二级索引的叶节点存储的是主键值（或者说聚簇索引的值），所以通过二级索引查询数据时，还需要将对应的主键去聚簇索引中再次进行查询。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208211352020.png)

接下来以两条SQL的执行为例，讲解一下InnoDB对于单行数据的加锁原理：

```sql
Copyupdate user set age = 10 where id = 49;
update user set age = 10 where name = 'Tom';
```

第一条SQL使用主键查询，只需要在 id = 49 这个主键索引上加上锁。第二条 SQL 使用二级索引来查询，那么首先在 name = Tom 这个索引上加写锁，然后由于使用 InnoDB 二级索引还需再次根据主键索引查询，所以还需要在 id = 49 这个主键索引上加锁。

也就是说使用主键索引需要加一把锁，使用二级索引需要在二级索引和主键索引上各加一把锁。

根据索引对单行数据进行更新的加锁原理了解了，那如果更新操作涉及多个行呢，比如下面 SQL 的执行场景。

```sql
Copyupdate user set age = 10 where id > 49;
```

上述 SQL 的执行过程如下图所示。MySQL Server 会根据 WHERE 条件读取第一条满足条件的记录，然后 InnoDB 引擎会将第一条记录返回并加锁，接着 MySQL Server 发起更新改行记录的 UPDATE 请求，更新这条记录。一条记录操作完成，再读取下一条记录，直至没有匹配的记录为止。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208211352512.png)

## 表锁

InnoDB既支持行锁，也支持表锁，当没有查询列没有索引时，InnoDB就不会去搞什么行锁了，毕竟行锁一定要有索引，所以它现在搞表锁，把整张表给锁住了。那么具体啥是表锁？还有其他什么情况下也会进行锁表呢？

表锁使用的是一次性锁技术，也就是说，在会话开始的地方使用 lock 命令将后续需要用到的表都加上锁，在表释放前，只能访问这些加锁的表，不能访问其他表，直到最后通过 unlock tables 释放所有表锁。

除了使用 unlock tables 显示释放锁之外，会话持有其他表锁时执行lock table 语句会释放会话之前持有的锁；会话持有其他表锁时执行 start transaction 或者 begin 开启事务时，也会释放之前持有的锁。

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111201516555.png)

表锁由 MySQL Server 实现，行锁则是存储引擎实现，不同的引擎实现的不同。在 MySQL 的常用引擎中 InnoDB 支持行锁，而 MyISAM 则只能使用 MySQL Server 提供的表锁。

## 两种锁的比较

表锁：加锁过程的开销小，加锁的速度快；不会出现死锁的情况；锁定的粒度大，发生锁冲突的几率大，并发度低；

- 一般在执行DDL语句时会对整个表进行加锁，比如说 ALTER TABLE 等操作；
- 如果对InnoDB的表使用行锁，被锁定字段不是主键，也没有针对它建立索引的话，那么将会锁整张表；
- 表级锁更适合于以查询为主，并发用户少，只有少量按索引条件更新数据的应用，如Web 应用。

行锁：加锁过程的开销大，加锁的速度慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高；

- 最大程度的支持并发，同时也带来了最大的锁开销。
- 在 InnoDB 中，除单个 SQL 组成的事务外，锁是逐步获得的，这就决定了在 InnoDB 中发生死锁是可能的。
- 行级锁只在存储引擎层实现，而 MySQL 服务器层没有实现。 行级锁更适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用，如一些在线事务处理（OLTP）系统。

# 3、MyISAM 表锁

## MyISAM 表级锁模式

- 表共享读锁（Table Read Lock）：不会阻塞其他线程对同一个表的读操作请求，但会阻塞其他线程的写操作请求；
- 表独占写锁（Table Write Lock）：一旦表被加上独占写锁，那么无论其他线程是读操作还是写操作，都会被阻塞；

默认情况下，写锁比读锁具有更高的优先级；当一个锁释放后，那么它会优先相应写锁等待队列中的锁请求，然后再是读锁中等待的获取锁的请求。

这种设定也是**MyISAM表不适合于有大量更新操作和查询操作**的原因。大量更新操作可能会造成查询操作很难以获取读锁，从而过长的阻塞。同时一些需要长时间运行的查询操作，也会使得线程“饿死”，应用中应尽量避免出现长时间运行的查询操作（在可能的情况下可以通过使用中间表等措施对SQL语句做一定的“分解”，使每一步查询都能在较短的时间内完成，从而减少锁冲突。如果复杂查询不可避免，应尽量安排在数据库空闲时段执行，比如一些定期统计可以安排在夜间执行。）

## MyISAM 对表加锁分析

MyISAM在执行查询语句（SELECT）前，会自动给涉及的所有表加读锁，在执行更新操作（UPDATE、DELETE、INSERT等）前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此用户一般不需要直接用 LOCK TABLE 命令给 MyISAM 表显式加锁。在自动加锁的情况下，MyISAM 总是一次获得 SQL 语句所需要的全部锁，这也正是 MyISAM 表不会出现死锁（Deadlock Free）的原因。

# 4、InnoDB 行锁与表锁

`InnoDB` 通过 `MVCC` 和 `NEXT-KEY Locks`，解决了在`可重复读`的事务隔离级别下出现`幻读`的问题。

## InnoDB 锁模式

1）InnoDB 中的行锁

InnoDB实现了以下两种类型的行锁：

- 共享锁（S）：加了锁的记录，所有事务都能去读取但不能修改，同时阻止其他事务获得相同数据集的排他锁；
- 排他锁（X）：允许已经获得排他锁的事务去更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁；

2）InnoDB 表锁——意向锁

由于表锁和行锁虽然锁定范围不同，但是会相互冲突。当你要加表锁时，势必要先遍历该表的所有记录，判断是否有排他锁。这种遍历检查的方式显然是一种低效的方式，MySQL引入了意向锁，来检测表锁和行锁的冲突。

意向锁也是表级锁，分为读意向锁（IS锁）和写意向锁（IX锁）。当事务要在记录上加上行锁时，要首先在表上加上意向锁。这样判断表中是否有记录正在加锁就很简单了，只要看下表上是否有意向锁就行了，从而就能提高效率。

意向锁之间是不会产生冲突的，它只会阻塞表级读锁或写锁。意向锁不与行级锁发生冲突。

## InnoDB 的加锁方法

- 意向锁是 InnoDB 自动加的，不需要用户干预；
- 对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及的数据集加上排他锁；
- 对于普通的SELECT语句，InnoDB不会加任何锁；事务可以通过以下语句显示给记录集添加共享锁或排他锁：
  - 共享锁（S）：`select * from table_name where ... lock in share mode`。此时其他 session 仍然可以查询记录，并也可以对该记录加 share mode 的共享锁。但是如果当前事务需要对该记录进行更新操作，则很有可能造成死锁。
  - 排他锁（X）：`select * from table_name where ... for update`。其他session可以查询记录，但是不能对该记录加共享锁或排他锁，只能等待锁释放后在加锁。

### select for update

在执行这个 select 查询语句的时候，会将对应的索引访问条目加上排他锁（X锁），也就是说这个语句对应的锁就相当于update带来的效果；

**使用场景**：为了让确保自己查找到的数据一定是最新数据，并且查找到后的数据值允许自己来修改，此时就需要用到select for update语句；

**性能分析**：select for update语句相当于一个update语句。在业务繁忙的情况下，如果事务没有及时地commit或者rollback可能会造成事务长时间的等待，从而影响数据库的并发使用效率。

### select lock in share mode

in share mode 子句的作用就是将查找的数据加上一个share锁，这个就是表示其他的事务只能对这些数据进行简单的 select 操作，而不能进行更改操作。

**使用场景**：为了确保自己查询的数据不会被其他事务正在修改，也就是确保自己查询到的数据是最新的数据，并且不允许其他事务来修改数据。与select for update不同的是，本事务在查找完之后不一定能去更新数据，因为有可能其他事务也对同数据集使用了 in share mode 的方式加上了S锁；

**性能分析**：select lock in share mode 语句是一个给查找的数据上一个共享锁（S 锁）的功能，它允许其他的事务也对该数据上S锁，但是不能够允许对该数据进行修改。如果不及时的commit 或者rollback 也可能会造成大量的事务等待。

## 行锁的类型

行锁进一步的划分：Next-Key Lock、Gap Lock、Record Lock以及插入意向GAP锁。

不同的锁锁定的位置是不同的，比如说记录锁只锁定对应的记录，而间隙锁锁住记录和记录之间的间隙，Next-key Lock则锁住所属记录之间的间隙。不同的锁类型锁定的范围大致如图所示：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208211352607.png)

### 记录锁（Record Lock）

记录锁最简单的一种行锁形式，上面我们以及稍微提及过了。这里补充下的点就是：行锁是加在索引上的，如果当你的查询语句不走索引的话，那么它就会升级到表锁，最终造成效率低下，所以在写SQL语句时需要特别注意。

### 间隙锁（Gap Lock）

> A gap lock is a lock on a gap between index records, or a lock on the gap before the first or after the last index record。

当我们使用范围条件而不是相等条件去检索，并请求锁时，InnoDB就会给符合条件的记录的索引项加上锁；而对于键值在条件范围内但并不存在（参考上面所说的空闲块）的记录，就叫做间隙，InnoDB在此时也会对间隙加锁，这种记录锁+间隙锁的机制叫Next-Key Lock。

从上面这句话可以表明间隙锁是所在两个存在的索引之间，是一个开区间，像最开始的那张索引图，15和18之间，是有（16，17）这个间隙存在的。

> Gap locks in InnoDB are “purely inhibitive”, which means that their only purpose is to prevent other transactions from inserting to the gap. Gap locks can co-exist. A gap lock taken by one transaction does not prevent another transaction from taking a gap lock on the same gap. There is no difference between shared and exclusive gap locks. They do not conflict with each other, and they perform the same function.

上面这段话表明间隙锁是可以共存的，共享间隙锁与独占间隙锁之间是没有区别的，两者之间并不冲突。其存在的目的都是防止其他事务往间隙中插入新的纪录，故而一个事务所采取的间隙锁是不会去阻止另外一个事务在同一个间隙中加锁的。

当然也不是在什么时候都会去加间隙锁的：

> Gap locking can be disabled explicitly. This occurs if you change the transaction isolation level to READ COMMITTED. Under these circumstances, gap locking is disabled for searches and index scans and is used only for foreign-key constraint checking and duplicate-key checking.

这段话表明，在 RU 和 RC 两种隔离级别下，即使你使用 select in share mode 或 select for update，也无法防止**幻读**（读后写的场景）。因为这两种隔离级别下只会有**行锁**，而不会有**间隙锁**。而如果是 RR 隔离级别的话，就会在间隙上加上间隙锁。

### 临键锁（Next-key Lock）

> A next-key lock is a combination of a record lock on the index record and a gap lock on the gap before the index record.

**临键锁是记录锁与与间隙锁的结合**，所以临键锁与间隙锁是一个同时存在的概念，并且临键锁是个左开右闭的却比如(16, 18]。

关于临键锁与幻读，官方文档有这么一条说明：

> By default, InnoDB operates in REPEATABLE READ transaction isolation level. In this case, InnoDB uses next-key locks for searches and index scans, which prevents phantom rows.

就是说 MySQL 默认隔离级别是RR，在这种级别下，如果你使用 select in share mode 或者 select for update 语句，那么InnoDB会使用临键锁（记录锁 + 间隙锁），因而可以防止幻读；

但是我也在网上看到相关描述：即使你的隔离级别是 RR，如果你这是使用普通的select语句，那么此时 InnoDB 引擎将是使用快照读，而不会使用任何锁，因而还是无法防止幻读。

### 插入意向锁（Insert Intention Lock）

> An insert intention lock is a type of gap lock set by INSERT operations prior to row insertion. This lock signals the intent to insert in such a way that multiple transactions inserting into the same index gap need not wait for each other if they are not inserting at the same position within the gap. Suppose that there are index records with values of 4 and 7. Separate transactions that attempt to insert values of 5 and 6, respectively, each lock the gap between 4 and 7 with insert intention locks prior to obtaining the exclusive lock on the inserted row, but do not block each other because the rows are nonconflicting.

官方文档已经解释得很清楚了，这里我做个翻译机：

插入意图锁是一种间隙锁，在行执行 INSERT 之前的插入操作设置。如果多个事务 INSERT 到同一个索引间隙之间，但没有在同一位置上插入，则不会产生任何的冲突。假设有值为4和7的索引记录，现在有两事务分别尝试插入值为 5 和 6 的记录，在获得插入行的排他锁之前，都使用插入意向锁锁住 4 和 7 之间的间隙，但两者之间并不会相互阻塞，因为这两行并不冲突。

插入意向锁只会和 间隙或者 Next-key 锁冲突，正如上面所说，间隙锁作用就是防止其他事务插入记录造成幻读，正是由于在执行 INSERT 语句时需要加插入意向锁，而插入意向锁和间隙锁冲突，从而阻止了插入操作的执行。

### 不同类型锁之间的兼容

不同类型的锁之间的兼容如下表所示：

|          | RECORED | GAP  | NEXT-KEY | II GAP（插入意向锁） |
| -------- | ------- | ---- | -------- | -------------------- |
| RECORED  |         | 兼容 |          | 兼容                 |
| GAP      | 兼容    | 兼容 | 兼容     | 兼容                 |
| NEXT-KEY |         | 兼容 |          | 兼容                 |
| II GAP   | 兼容    |      |          | 兼容                 |

（其中行表示已有的锁，列表示意图加上的锁）

其中，第一行表示已有的锁，第一列表示要加的锁。插入意向锁较为特殊，所以我们先对插入意向锁做个总结，如下：

- 插入意向锁不影响其他事务加其他任何锁。也就是说，一个事务已经获取了插入意向锁，对其他事务是没有任何影响的；
- 插入意向锁与间隙锁和 Next-key 锁冲突。也就是说，一个事务想要获取插入意向锁，如果有其他事务已经加了间隙锁或 Next-key 锁，则会阻塞。

其他类型的锁的规则较为简单：

- 间隙锁不和其他锁（不包括插入意向锁）冲突；
- 记录锁和记录锁冲突，Next-key 锁和 Next-key 锁冲突，记录锁和 Next-key 锁冲突；
