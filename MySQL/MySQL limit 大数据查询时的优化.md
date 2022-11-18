参考 https://www.cnblogs.com/weixiaotao/p/10646666.html

# 耗时本质

mysql大数据量使用limit分页，随着页码的增大，查询效率越低下。

**当一个表数据有几百万的数据的时候成了问题！**

如 select * from table limit 0,10 这个没有问题 当 limit 200000,10 的时候数据读取就很慢

原因本质： 1）limit语句的查询时间与起始记录（offset）的位置成正比 2）mysql的limit语句是很方便，但是对记录很多:百万，千万级别的表并不适合直接使用。

例如： limit10000,20的意思扫描满足条件的10020行，扔掉前面的10000行，返回最后的20行，问题就在这里。  LIMIT 2000000, 30 扫描了200万+ 30行，怪不得慢的都堵死了，甚至会导致磁盘io 100%消耗。  但是: limit 30 这样的语句仅仅扫描30行。

# 优化手段

不是直接使用limit，而是首先获取到offset的id然后直接使用limit size来获取数据

利用表的覆盖索引来加速分页查询

覆盖索引:

就是select 的数据列只用从索引中就能获得，不必读取数据行。mysql 可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，换句话说：**查询列要被所创建的索引覆盖**

```sql
# 可得到 200001 - 2000020 数据 id
select id from order_table where company_id = 1 and mark =0 order by id desc limit 200000 ,20;
# 这个便可通过覆盖索引的方法，得到 id 后，再使用索引查找
SELECT * FROM order_table WHERE ID > =(select id from order_table limit 200000, 1) limit 20;
```

