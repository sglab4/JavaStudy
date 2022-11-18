# 1、概念

NOSQU 系列的非关系型数据库。

关系型数据库：

1. 数据之间有关系
2. 数据存储在硬盘文件上

非关系型数据库：

1. 数据之间没有关系
2. 数据存储在内存中

对于存储在关系型数据库中的大规模数据来说，用户想要访问一个或几个数据，会非常耗时，针对经常访问的不太发生变化的数据，引入了非关系型数据库，采用缓存的思想解决这个问题。方法如下：

当用户访问数据时，首先在缓存中查询数据，若有，则直接返回，若无，则从关系型数据库中查询数据，放入缓存，下次访问时直接返回即可。

参考 https://blog.csdn.net/Lucien010230/article/details/116301713 

## 关系数据库与非关系型数据库概述

### 1、关系型数据库

-  关系型数据库是一个结构化的数据库，创建在关系模型（二维表格模型）基础上，一般面向于记录。、
-  SQL 语句（标准数据查询语言）就是一种基于关系型数据库的语言，用于执行对关系型数据库中数据的检索和操作。
-  主流的关系型数据库包括 Oracle、MySQL、SQL Server、Microsoft Access、DB2 等。

### 2、非关系型数据库

- NoSQL(NoSQL = Not Only SQL )，意思是“不仅仅是 SQL”，是非关系型数据库的总称。
- 除了主流的关系型数据库外的数据库，都认为是非关系型。
- 主流的 NoSQL 数据库有 Redis、MongDB、Hbase、Memcached 等。

### 3、关系数据库与非关系型数据库区别

1、数据存储方式不同

- 关系型和非关系型数据库的主要差异是数据存储的方式。关系型数据天然就是表格式的，因此存储在数据表的行和列中。数据表可以彼此关联协作存储，也很容易提取数据。
- 与其相反，非关系型数据不适合存储在数据表的行和列中，而是大块组合在一起。非关系型数据通常存储在数据集中，就像文档、键值对或者图结构。你的数据及其特性是选择数据存储和提取方式的首要影响因素。

2、扩展方式不同

- SQL和NoSQL数据库最大的差别可能是在扩展方式上，要支持日益增长的需求当然要扩展。
- 要支持更多并发量，SQL数据库是纵向扩展，也就是说提高处理能力，使用速度更快速的计算机，这样处理相同的数据集就更快了。因为数据存储在关系表中，操作的性能瓶颈可能涉及很多个表，这都需要通过提高计算机性能来客服。虽然SQL数据库有很大扩展空间，但最终肯定会达到纵向扩展的上限。
- 而NoSQL数据库是横向扩展的。因为**非关系型数据存储天然就是分布式的**，NoSQL数据库的扩展可以通过给资源池添加更多普通的数据库服务器(节点)来分担负载。

## Redis 简介

- Redis 是一个开源的、使用 C 语言编写的 NoSQL 数据库。
- Redis 基于内存运行并支持持久化，采用 key-value（键值对）的存储形式，是目前分布式架构中不可或缺的一环。

### 1、Redis的单线程模式

- Redis服务器程序是单进程模型，也就是在一台服务器上可以同时启动多个Redis进程，Redis的实际处理速度则是完全依靠于主进程的执行效率。
- 若在服务器上只运行一个Redis进程，当多个客户端同时访问时，服务器的处理能力是会有一定程度的下降
- 若在同一台服务器上开启多个Redis进程，Redis在提高并发处理能力的同时会给服务器的CPU造成很大压力。
- 在实际生产环境中，需要根据实际的需求来决定开启多少个Redis进程。若对高并发要求更高一些，可能会考虑在同一台服务器上开启多个进程。若 CPU 资源比较紧张，采用单进程即可。

### 2、Redis的优点

- 具有极高的数据读写速度
- 支持丰富的数据类型
- 支持数据的持久化
- 原子性
- 支持数据备份

# 2、安装

Redis 下载解压后，需要使用 Redis 的客户端访问 Redis 的服务器端，服务器端默认端口为 6379，PID 为 7700

![image-20210910152017200](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210910152017.png)

服务器端：

![image-20210910151724285](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210910151740.png)

客户端：

![image-20210910151917801](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210910151918.png)

# 3、命令操作

## 1、Redis 的数据结构

Redis 存储的都是 key-value 类型的数据，其中 key 均为字符串，value 有五种数据结构，为

- string ：字符串（可以为整型、浮点型和字符串，通称为元素）
- list ：列表（实现队列，元素不唯一，先入先出原则）
- set ：集合（各不相同的元素）
- hash ：hash 散列值（hash 的 key 必须是唯一的，可理解为 Map 格式）
- sorted set ：有序集合

## 2、命令

> string 类型：

- 存储：set key value
- 获取：get key
- 删除：del key

> hash 类型：

- 存储：hset key field value

- 获取：hget key field，获取单一的 field 对应的值

  ​			hgetall key，获取所有 field 对应的值

- 删除：hdel key field

> list 类型：

- 存储：lpush key value，从列表左边加入

  ​			rpush key value，从列表右边加入

- 获取：lrange key start end：范围获取，若想返回所有数据，则应该是 lrang key 0 -1，Redis 也支持负索引，-1 表示右边第一个元素，0 表示左边第一个元素。若 end 大于实际索引返回，Redis 则会返回到列表最右边的元素。			  

- 删除：lpop key：删除列表最左边的元素并返回

  ​			rpop key：删除列表最右边的元素并返回

- 列表元素个数：llen key

> set 类型：不允许重复元素，不保证顺序，即存入和取出顺序不同

- 存储：sadd key value
- 获取：smembers key，获取集合 key 中所有元素
- 删除：srem key value，删除集合中的某个元素

> sorted set 类型：

- 存储：zadd key score value，value 会根据 score 进行排序

- 获取：zrange key start end，获取 value

  ​			zrange key start end withscores，获取了 value 和 score

- 删除：zrem key value

## 3、通用命令

keys * （* 也可用正则表达式）：查询所有键

type key：查询 key 中存储的 value 的类型

## 4、持久化

Redis 数据存储在内存中，当 Redis 服务器关闭或重启，数据将会丢失，因此可将 Redis 数据持久化到硬盘中

### 4.1 快照（RDB）

Redis 使用操作系统的多进程 COW (Copy On Write) 机制实现快照持久化。在持久化时，由于要一边要持久化，一边又要满足 Redis 的正常使用，所以 Redis 在持久化的时候，使用了 glibc 的 **fork 函数产生了一个子进程**，在子进程中进行快照持久化操作，主进程满足业务的正常使用。

RDB的原理是 **fork 和 cow** 。fork是指redis通过创建子进程来进行RDB操作，cow指的是**copy on write**，子进程创建后，父子进程共享数据段，父进程继续提供读写服务，写脏的页面数据会逐渐和子进程分离开来。

快照持久化的内存策略是：子进程按照**数据页**进行复制。在持久化过程中，子进程负责持久化过程，它将主进程数据段的数据页复制一份出来，然后对原数据页进行存储；同一时间，主进程对那份复制出来的数据进行操作。通过复制小数据量数据页（通常每个数据页只有 4KB）的方式，保证了主进程修改数据不会影响子进程存储。

快照触发方式有自动触发与手动触发两种。

- **自动触发**：通过 redis.conf 配置文件进行配置；
- 手动触发
  - save: 阻塞式触发，在未完成前，客户端无法进行命令操作；
  - bgsave: 非阻塞式触发，主进程 fork 出子进程进行备份操作；

在恢复文件时，将备份文件 (dump.rdb) 放在 Redis 安装目录，然后启动 Redis，就能把 RDB 中的文件加载到 Redis 服务中。

快照存储的优点缺点：

- 优点
  - 数据结构紧凑，保存了 Redis 服务在某个时间点上的数据集，非常适合做备份和灾难恢复；
  - 进行 RDB 快照持久化时，主进程会 fork 出一个子进程进行备份工作，主进程不需要额外的 IO 操作；
  - RDB 恢复大数据集时，速度比 AOF 快；
- 缺点
  - 版本兼容问题：Redis 版本更新过程中有多个 RDB 版本，存在老版 RDB 兼容性无法兼容新版的问题；
  - 无法做到实时持久化，因为 bgsave 时的 fork 操作每次都会创建子进程，内存中的数据被克隆了一份，属于重量级操作。

### 4.2 AOF

AOF 机制默认关闭 

```conf
# 每次操作都存储
appendfsync always
# 每个一秒存储一次
appendfsync everysec
# 关闭
appendonly no
```

AOF 日志存储的是 **Redis 服务器的顺序指令序列**，它只记录对内存进行修改的指令记录。AOF 使用追加记录的方式，在 Redis 长期运行的过程中，AOF 日志会越来越长，所以一旦宕机重启载入 AOF 日志，将会是一个非常耗时的功能，因此我们需要对 AOF 进行瘦身，即 AOF 重写。

由于AOF持久化是Redis不断将写命令记录到 AOF 文件中，随着Redis不断的进行，AOF 的文件会越来越大，文件越大，占用服务器内存越大以及 AOF 恢复要求时间越长。为了解决这个问题，Redis新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。可以使用命令 bgrewriteaof 来重新。

AOF 重写并不是对原始的 AOF 文件进行重新整理，而是 fork 一个子进程遍历服务器的键值对，转换成一系列 Redis 的操作指令，序列化到一个新的 AOF 日志文件中。序列化完毕后，将原来的文件替换为序列化后的文件即可。 

AOF 重写分为两个过程，一个是 fork 子进程重写过程执行的原始数据内容，另一个是在 **fork 过程中主进程修改的指令**。

# 4、Jedis

参考 https://zhuanlan.zhihu.com/p/114513320

## 4.1 配置

1、下载 jar 包

下载地址：[https://mvnrepository.com/artif](https://link.zhihu.com/?target=https%3A//mvnrepository.com/artifact/redis.clients/jedis)

2、在 maven 中添加依赖坐标

```xml
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

## 4.2 简单示例

```java
Jedis jedis = new Jedis("localhost", 6379); //若使用空参构造，默认为 localhost 和 6379 端口，可通过源码查看
// 尝试操作Redis
jedis.set("msg", "Hello World!");
String msg = jedis.get("msg");	
System.out.println(msg);	// 打印"Hello World!"
// 关闭Redis连接
jedis.close();
```

## 4.3 连接池

JedisPool

使用步骤：

1. 创建连接池，new JedisPool
2. 获取连接，JedisPool.getResoource()

jedis 直接使用 apache common-pool2 的 GenericObjectPool 来管理连接池。

JedisPool 对象继承自 JedisPoolAbstract，然后 JedisPoolAbstract 继承自 Pool，然后 Pool 中包含 protected 成员 GenericObjectPool internalPool。

Pool 的构造函数取 final GenericObjectPoolConfig poolConfig 和 PooledObjectFactory factory 两个参数，大致上直接透传给了 GenericObjectPool，poolConfig 用于指定 maxIdle、maxAlive 等连接池常见参数，在创建 JedisPool 时未指定参数 Config 时，将会采用 GenericObjectPoolConfig 中的默认参数，而 PooledObjectFactory 这里的实现来自 JedisFactory，用于生产 Jedis 对象，每个 Jedis 对象对应一个连接。

可采用 JedisPoolConfig 配置参数，在 JedisPool 的构造方法中，有

```java
JedisPool(GenericObjectPoolConfig poolConfig, String host, int port)
```

![image-20210910182635220](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210910182651.png)

## 4.4 连接池工具类

用来加载配置文件

Jedis 的 properties 配置文件如下

```
host=127.0.0.1
port=6379
MaxTotal=50
MaxIdle=10
```

JedisPoolUntil.java 配置类如下：

```java
package com.jedis;

import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

/**
 * Jedis 工具类
 */
public class JedisPoolUtil {
    private static JedisPool jedisPool;

    static {
        InputStream is = JedisPoolUtil.class.getResourceAsStream("jedis.properties");
        Properties properties = new Properties();
        try {
            properties.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //获取数据
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(Integer.getInteger(properties.getProperty("MaxTotal")));
        config.setMaxIdle(Integer.getInteger(properties.getProperty("MaxIdle")));
        jedisPool = new JedisPool(config, properties.getProperty("host"), Integer.getInteger(properties.getProperty("port")));
    }

    public JedisPool jedisPool() {
        return jedisPool;
    }
}
```

# 5、Redis 内存淘汰机制

|      策略       |                         描述                         |
| :-------------: | :--------------------------------------------------: |
|  volatile-lru   | 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰 |
|  volatile-ttl   |   从已设置过期时间的数据集中挑选将要过期的数据淘汰   |
| volatile-random |      从已设置过期时间的数据集中任意选择数据淘汰      |
|   allkeys-lru   |       从所有数据集中挑选最近最少使用的数据淘汰       |
| allkeys-random  |          从所有数据集中任意选择数据进行淘汰          |
|   noeviction    |                     禁止驱逐数据                     |
|   allkeys-lfu   |              使用 LFU 算法进行筛选删除               |
|  volatile-lfu   |              使用 LFU 算法进行筛选删除               |

默认的淘汰策略是 noevition，也就是不淘汰

设置淘汰策略的方法：

1. 命令行：config set maxmemory-policy allkeys-lru

2. 配置文件

   ![image-20211007155119692](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211007155119.png)

**LRU 算法**

LRU 全称是 Least Recently Used，即最近最少使用，会将最不常用的数据筛选出来，保留最近频繁使用的数据。

LRU 会把所有数据组成一个链表，链表头部称为 MRU，代表最近最常使用的数据；尾部称为 LRU代表最近最不常使用的数据；

![image-20211007155403697](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211007155403.png)

LRU 算法在实现过程中使用链表管理所有缓存的数据，这会给 Redis 带来额外的开销，而且，当有数据访问时就会有链表移动操作，进而降低 Redis 的性能。

于是，Redis 对 LRU 的实现进行了一些改变：

- 记录每个 key 最近一次被访问的时间戳（由键值对数据结构 RedisObject 中的 lru 字段记录）
- 在第一次淘汰数据时，会先随机选择 N 个数据作为一个候选集合，然后淘汰 lru 值最小的。（N 可以通过 `config set maxmemory-samples 100` 命令来配置）
- 后续再淘汰数据时，会挑选数据进入候选集合，进入集合的条件是：它的 lru 小于候选集合中最小的 lru。
- 如果候选集合中数据个数达到了 maxmemory-samples，Redis 就会将 lru 值小的数据淘汰出去。

**LFU 算法**

LFU 全称 Least Frequently Used，即最不经常使用策略，它是基于数据访问次数来淘汰数据的，在 Redis 4.0 时添加进来。它在 LRU 策略基础上，为每个数据增加了一个计数器，来统计这个数据的访问次数。

前面说到，LRU 使用了 RedisObject 中的 lru 字段记录时间戳，lru 是 24bit 的，LFU 将 lru 拆分为两部分：

- ldt 值：lru 字段的前 16bit，表示数据的访问时间戳
- counter 值：lru 字段的后 8bit，表示数据的访问次数

使用 LFU 策略淘汰缓存时，会把访问次数最低的数据淘汰，如果访问次数相同，再根据访问的时间，将访问时间戳最小的淘汰。

**为什么 Redis 有了 LRU 还需要 LFU 呢？**

在一些场景下，有些数据被访问的次数非常少，甚至只会被访问一次。当这些数据服务完访问请求后，如果还继续留存在缓存中的话，就只会白白占用缓存空间。

由于 LRU 是基于访问时间的，如果系统对大量数据进行单次查询，这些数据的 lru 值就很大，使用 LFU 算法就不容易被淘汰。

# 6、Redis 事件机制

Redis程序的运行过程是一个处理事件的过程，也称Redis是一个事件驱动的服务。Redis中的事件分两类：文件事件（File Event）、时间事件（Time Event）。文件事件处理文件的读写操作，特别是与客户端通信的 Socket 文件描述符的读写操作；时间事件主要用于处理一些定时处理的任务。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211007160105.jpeg)

## 文件事件

服务器通过套接字与客户端或者其它服务器进行通信，文件事件就是对套接字操作的抽象。

Redis 基于 Reactor 模式开发了自己的网络事件处理器，使用 I/O 多路复用程序来同时监听多个套接字，并将到达的事件传送给文件事件分派器，分派器会根据套接字产生的事件类型调用相应的事件处理器。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211007160237.png)

- 客户端向服务端发起建立 socket 连接的请求，那么监听套接字将产生 AE_READABLE 事件，触发**连接应答处理器**执行。处理器会对客户端的连接请求进行应答，然后创建客户端套接字，以及客户端状态，并将客户端套接字的 AE_READABLE 事件与**命令请求处理器**关联。
- 客户端建立连接后，向服务器发送命令，那么客户端套接字将产生 AE_READABLE 事件，触发**命令请求处理器**执行，处理器读取客户端命令，然后传递给相关程序去执行。
- 执行命令获得相应的命令回复，为了将命令回复传递给客户端，服务器将客户端套接字的 AE_WRITEABLE 事件与**命令回复处理器**关联。当客户端试图读取命令回复时，客户端套接字产生 AE_WRITEABLE 事件，触发命令**回复处理器**将命令回复全部写入到套接字中。

## 事件事件

服务器有一些操作需要在给定的时间点执行，时间事件是对这类定时操作的抽象。

时间事件又分为：

- 定时事件：是让一段程序在指定的时间之内执行一次；
- 周期性事件：是让一段程序每隔指定时间就执行一次。

Redis 将所有时间事件都放在一个无序链表中，通过遍历整个链表查找出已到达的时间事件，并调用相应的事件处理器。



























