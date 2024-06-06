## 数据倾斜

​	绝大部分任务都完成，只有一个或少数几个任务执行很慢，甚至最终失败。

需要分清，单个 Key 还是多个 Key 的倾斜

### 单表数据倾斜



是否在 Map 端进行聚合，默认为 True

```sql
set hive.map.aggr=true;
```

在 Map 端进行聚合操作的条目数目

```sql
set hive.groupby.mapaggr.checkinterval = 100000;
```



有数据倾斜时，进行负载均衡（默认是 false）

```sql
set hive.groupby.skewindata = true;
```



### 增加Reduce数量

解决多个 Key 同时导致数据倾斜，增加 reduce 数量，减少多个reduce的差距，增加的数量最好不要使用倍数，原本是2个，优化时，使用 5、7个这种。

每个 reduce 默认数据量

```sql
set hive.exec.reducers.bytes.per.reducer = 256000000;
```

每个任务最大 reduce数，默认 1009

```sql
set hive.exec.reducer.max = 1009
```

计算 recuer 数公式：参数2是1009，参数1是256M

```sql
N = min(参数2，总输入数据量/参数1)
```

调整 reduce 个数

在 hadoop 的 `mapred-default.xml` 中调整

```sql
set mapreduce.job.reduces = 15;
```



### Join数据倾斜优化

