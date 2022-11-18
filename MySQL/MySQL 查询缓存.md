参考 https://www.jianshu.com/p/5c3ddf9a454c

开启查询缓存后，查询语句的解析过程：

在解析一个查询语句之前，如果查询缓存是打开的，那么MySQL会优先检查这个查询是否命中查询缓存中的数据。如果当前的查询恰好命中了查询缓存，那么在返回查询结果之前MySQL会检查一次用户权限。若权限没有问题，MySQL会跳过所有其他阶段（解析、优化、执行等），直接从缓存中拿到结果并返回给客户端。这种情况下，查询不会被解析，不用生成执行计划，不会被执行。

# 开启查询缓存

使用 **query_cache_type** 变量来开启查询缓存，开启方式有三种：

- **ON** : 正常缓存。表示在使用 SELECT 语句查询时，若没指定 SQL_NO_CACHE 或其他非确定性函数，则一般都会将查询结果缓存下来。

- **DEMAND** ：指定SQL_CACHE才缓存。表示在使用 SELECT 语句查询时，必须在该 SELECT 语句中指定 SQL_CACHE 才会将该SELECT语句的查询结果缓存下来。

  > 例如：select SQL_CACHE name from user where id = 15;    #只有明确指定  SQL_CACHE 的SELECT语句，才会将查询结果缓存。

- **OFF**： 关闭查询缓存。

修改 my.cnf :

```undefined
query_cache_type = ON
```

重启 mysql 服务

# 设置查询缓存的大小

query_cache_size ：查询缓存的总体可用空间。

修改 my.cnf :

```bash
query_cache_size = 500M   #支持单位：K,M,G
```

重启 mysql 服务