参考 

- https://www.jianshu.com/p/20e10ed721d0
- https://www.cnblogs.com/dh17/articles/14919607.html

bin log、redo log、undo log三种日志属于不同级别的日志，按照mysql的划分可以分为服务层和引擎层两大层，bin log是在服务层实现的；redo log、undo log是在引擎层实现的，且是innodb引擎独有的，主要和事务相关。

# Bin log

bin log中记录的是整个mysql数据库的操作内容，对所有的引擎都适用，包括执行的DDL、DML，可以用来进行**数据库的恢复及复制**。bin log有三种形式：statement、row、mixed，statement是基于语句的，也就是执行的sql语句，该种形式的文件比较小，例，update t1 set age='24' where name like '%王%'，这样一条语句，在statement下就会记录这样一条sql；row是基于数据行的，会记录变化的所有数据，一般文件较大。例，update t1 set age='24' where name like '%王%'，这条语句，在row的形式下，则会记录该条sql影响的所有数据记录；mixed是混合格式，是statement和row的组合；

事务有4种特性：原子性、一致性、隔离性和持久性，在事务中的操作，要么全部执行，要么全部不做，这就是事务的目的。事务的隔离性由锁机制实现，原子性、一致性和持久性由事务的redo 日志和undo 日志来保证，bin log 和 redo log 实现事务的一致性。

# Redo log

## Redo 类型

重做日志(redo log)用来保证事务的持久性，即事务ACID中的D。实际上它可以分为以下两种类型：

- 物理Redo日志
- 逻辑Redo日志

在InnoDB存储引擎中，**大部分情况下 Redo 是物理日志，记录的是数据页的物理变化**。而逻辑Redo日志，不是记录页面的实际修改，而是记录修改页面的一类操作，比如新建数据页时，需要记录逻辑日志。关于逻辑Redo日志涉及更加底层的内容，这里我们只需要记住绝大数情况下，Redo是物理日志即可，DML对页的修改操作，均需要记录Redo.

## Redo 作用

Redo log的主要作用是用于数据库的崩溃恢复

## Redo 组成

Redo log可以简单分为以下两个部分：

- 一是内存中重做日志缓冲 (redo log buffer),是易失的，在内存中
- 二是重做日志文件 (redo log file)，是持久的，保存在磁盘中

## Redo 的写入流程

写入Redo的时机：

- 先对内存中的数据进行修改，然后修改日志内容
- **先将日志内容写入磁盘，再将内存中的内容写入磁盘**
- 聚集索引、二级索引、undo页面的修改，均需要记录Redo日志。

下面以一个更新事务为例，宏观上把握redo log 流转过程，如下图所示：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202111201633350.webp)

第一步：先将原始数据从磁盘中读入内存中来，修改数据的内存拷贝

第二步：生成一条重做日志并写入redo log buffer，记录的是**数据被修改后的值**

第三步：当事务commit时，将redo log buffer中的内容刷新到 redo log file，对 redo log file采用追加写的方式

第四步：定期将内存中修改的数据刷新到磁盘中

## Redo 如何保证事务的持久性？

InnoDB是事务的存储引擎，其通过**Force Log at Commit 机制**实现事务的持久性，即当事务提交时，先将 redo log buffer 写入到 redo log file 进行持久化，待事务的commit操作完成时才算完成。这种做法也被称为 **Write-Ahead Log(预先日志持久化)**，**在持久化一个数据页之前，先将内存中相应的日志页持久化**。

为了保证每次日志都写入redo log file，在每次将redo buffer写入redo log file之后，默认情况下，InnoDB存储引擎都需要调用一次 **fsync操作**,因为重做日志打开并没有 O_DIRECT选项，所以重做日志先写入到文件系统缓存。为了确保重做日志写入到磁盘，必须进行一次 fsync操作。fsync是一种系统调用操作，其fsync的效率取决于磁盘的性能，因此磁盘的性能也影响了事务提交的性能，也就是数据库的性能。

 **(O_DIRECT选项是在Linux系统中的选项，使用该选项后，对文件进行直接IO操作，不经过文件系统缓存，直接写入磁盘)**

上面提到的**Force Log at Commit机制**就是靠InnoDB存储引擎提供的参数 `innodb_flush_log_at_trx_commit`来控制的，该参数可以控制 redo log刷新到磁盘的策略，设置该参数值也可以允许用户设置非持久性的情况发生，具体如下：

- 当设置参数为1时，（默认为1），表示事务提交时必须调用一次 `fsync` 操作，最安全的配置，保障持久性
- 当设置参数为2时，则在事务提交时只做 **write** 操作，只保证将redo log buffer写到**系统的页面缓存**中，不进行fsync操作，因此如果MySQL数据库宕机时 不会丢失事务，但操作系统宕机则可能丢失事务
- 当设置参数为0时，表示事务提交时不进行写入redo log操作，这个操作仅在master thread 中完成，而在master thread中每1秒进行一次重做日志的fsync操作，因此实例 crash 最多丢失1秒钟内的事务。（master thread是负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性）

`fsync`和`write`操作实际上是系统调用函数，在很多持久化场景都有使用到，比如 Redis 的AOF持久化中也使用到两个函数。`fsync`操作 将数据提交到硬盘中，强制硬盘同步，将一直阻塞到写入硬盘完成后返回，大量进行`fsync`操作就有性能瓶颈，而`write`操作将数据写到系统的页面缓存后立即返回，后面依靠系统的调度机制将缓存数据刷到磁盘中去,其顺序是user buffer——> page cache——>disk。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202111201644603.webp)

**除了上面谈到的Force Log at Commit机制保证事务的持久性，实际上重做日志的实现还要依赖于mini-transaction。**

## Redo 在 InnoDB 中是如何实现的？与 mini-transaction 的联系？

Redo的实现则跟mini-transaction紧密相关，mini-transaction是一种InnoDB内部使用的机制，通过mini-transaction来**保证并发事务操作下以及数据库异常时数据页中数据的一致性**，但它不属于事务。

**为了使得mini-transaction保证数据页数据的一致性，mini-transaction必须遵循以下三种协议**：

- The FIX Rules
- Write-Ahead Log
- Force-log-at-commit

**The FIX Rules**

修改一个数据页时需要获得该页的x-latch(排他锁)，获取一个数据页时需要该页的s-latch(读锁或者称为共享锁) 或者是 x-latch，持有该页的锁直到修改或访问该页的操作完成。

**Write-Ahead Log**

在前面阐述中就提到了Write-Ahead Log(预先写日志)。在持久化一个数据页之前，必须先将内存中相应的日志页持久化。每个页都有一个LSN(log sequence number)，代表日志序列号，（LSN占用8字节，单调递增), 当一个数据页需要写入到持久化设备之前，要求内存中小于该页LSN的日志先写入持久化设备

那为什么必须要先写日志呢？可不可以不写日志，直接将数据写入磁盘？原则上是可以的，只不过会产生一些问题，数据修改会产生随机IO，但日志是顺序IO，append方式顺序写，是一种串行的方式，这样才能充分利用磁盘的性能。

**Force-log-at-commit**

这一点也就是前文提到的如何保证事务的持久性的内容，这里再次总结一下，与上面的内容相呼应。在一个事务中可以修改多个页，Write-Ahead Log 可以保证单个数据页的一致性，但是无法保证事务的持久性，Force-log-at-commit 要求当一个事务提交时，其产生所有的mini-transaction 日志必须刷新到磁盘中，若日志刷新完成后，在缓冲池中的页刷新到持久化存储设备前数据库发生了宕机，那么数据库重启时，可以通过日志来保证数据的完整性。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202111201646698.webp)

上图表示了重做日志的写入流程，每个mini-transaction对应每一条DML操作，比如一条update语句，其由一个mini-transaction来保证，对数据修改后，产生redo1，首先将其写入mini-transaction私有的Buffer中，update语句结束后，将redo1从私有Buffer拷贝到公有的Log Buffer中。当整个外部事务提交时，将redo log buffer再刷入到redo log file中。

## SQL 语句的两阶段提交

以更新 `update T set c=c+1 where ID=2;` 语句为例：

1. 执行器通过 InooDB 引擎去 ID 所在行，ID 为主键。引擎通过树搜索找到该行，如果该行所在数据页在内存中，返回给执行器。否则先从磁盘读入内存，然后再返回。
2. 执行器拿到引擎给的数据，将 C 值加 1，等到新的一行，然后通过引擎接口重新写入新数据。
3. 引擎将该行更新到内存中，同时将该更新操作记录到 **redo log** 中，并更改 redo log 的状态为 prepare 状态。然后告知执行器，在合适的时间提交事务。
4. 执行器生成这个操作的 **bin log**，并将 bin log 写入磁盘。
5. 执行器调用引擎到的提交事务接口，将刚刚写入的 redo log 改成 commit 状态，更新完成。

在更新内存后，将写入 redo log 拆分了成两个步骤：prepare 和 commit，就是常说的**两阶段提交，用于保证当有意外情况发生时，数据的一致性。**

不使用两阶段提交可能会造成数据不一致的问题。

> 不使用两阶段提交的后果

1. 先写 redo log，后写 bin log。在写入 redo log 后，未写入 binlog，此时若数据库发生异常，当再次启动数据库后，由于 redo log 正常，数据库内容没有问题，若想使用 bin log 进行备份，则会出现数据不一致的问题。
2. 先写 bin log，后写 redo log。写入 bin log 而未写入 redo log 时，若数据库发生异常，当再次启动数据库后，由于 redo log未写入该数据，认为该数据无效，但是在 bin log 中多了一条数据，造成数据不一致的问题。

> 两阶段提交的过程

1. 在写 redo log prepare 阶段崩溃，时刻 A 的位置。重启后，发现 redo log 没写入，回滚此次事务。
2. 如果在写 binlog 时奔溃，重启后，发现 binlog 未被写入，回滚操作。
3. binlog 写完，但在提交 redo log 的 commit 状态时发生崩溃.
   1. 如果 redo log 中事务完整，有了 commit 标识，直接提交。
   2. 如果 redo log 中只有完整的 prepare, 判断对应 binlog 是否完整。
      1. 完整，提交事务
      2. 不完整，回滚事务。

# Undo log

## Undo log 的定义

undo log主要记录的是数据的**逻辑变化**，为了在发生错误时回滚之前的**操作**，需要将之前的操作都记录下来，然后在发生错误时才可以回滚。

## Undo log 的作用

undo是一种**逻辑日志**，有两个作用：

- 用于事务的回滚
- MVCC

关于MVCC(多版本并发控制)的内容这里就不多说了，本文重点关注undo log用于事务的回滚。

undo日志，只将数据库逻辑地恢复到原来的样子，**在回滚的时候，它实际上是做的相反的工作**，比如一条INSERT ，对应一条 DELETE，对于每个UPDATE,对应一条相反的 UPDATE,将修改前的行放回去。undo日志用于事务的回滚操作进而保障了事务的原子性。

## Undo log 的写入时机

- DML操作修改聚簇索引前，记录undo日志
- 二级索引记录的修改，不记录undo日志

需要注意的是，undo页面的修改，同样需要记录redo日志。

## Undo 的存储位置

在InnoDB存储引擎中，undo存储在回滚段(Rollback Segment)中,每个回滚段记录了1024个undo log segment，而在每个undo log segment段中进行undo 页的申请，在5.6以前，Rollback Segment是在共享表空间里的，5.6.3之后，可通过 innodb_undo_tablespace设置undo存储的位置。

## Undo 的类型

在InnoDB存储引擎中，undo log分为：

- insert undo log
- update undo log

insert undo log是指在insert 操作中产生的undo log，因为insert操作的记录，只对事务本身可见，对其他事务不可见。故该undo log可以在事务提交后直接删除，不需要进行purge操作。

而update undo log记录的是对delete 和update操作产生的undo log，该undo log可能需要提供MVCC机制，因此不能再事务提交时就进行删除。提交时放入undo log链表，等待purge线程进行最后的删除。

补充：purge线程两个主要作用是：清理undo页和清除page里面带有Delete_Bit标识的数据行。在InnoDB中，事务中的Delete操作实际上并不是真正的删除掉数据行，而是一种Delete Mark操作，在记录上标识Delete_Bit，而不删除记录。是一种"假删除",只是做了个标记，真正的删除工作需要后台purge线程去完成。

## Undo log 是否是 Redo log 的逆过程？

undo log 是否是redo log的逆过程？其实从前文就可以得出答案了，undo log是逻辑日志，对事务回滚时，只是将数据库逻辑地恢复到原来的样子，而redo log是物理日志，记录的是数据页的物理变化，显然undo log不是redo log的逆过程。

# Redo & Undo

由于 Undo 记录的是逆过程，因此记录的是 A = 1 和 B = 2

```html
假设有A、B两个数据，值分别为1,2.
1. 事务开始
2. 记录A=1到undo log
3. 修改A=3
4. 记录A=3到 redo log
5. 记录B=2到 undo log
6. 修改B=4
7. 记录B=4到redo log
8. 将redo log写入磁盘
9. 事务提交
```

# Bin log

作用：用于复制，在主从复制中，从库利用主库上的 binlog 进行重播，实现主从同步。 用于数据库的基于时间点的还原

内容：逻辑格式的日志，可以简单认为就是执行过的事务中的 sql 语句。但又不完全是 sql 语句这么简单，而是包括了执行的 sql 语句（增删改）反向的信息，也就意味着 delete 对应着 delete 本身和其反向的 insert；update 对应着 update 执行前后的版本的信息；insert 对应着 delete 和 insert 本身的信息。

binlog 有三种模式：Statement（基于 SQL 语句的复制）、Row（基于行的复制） 以及 Mixed（混合模式）

bin log 和 redo log 的区别：

1. redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用
2. redo log 是物理日志，记录的是 “在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如 “给 ID=2 这一行的 c 字段加 1 ”
3. redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写” 是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志

