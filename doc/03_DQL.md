## HQL语法优化

### Group By

默认情况下，Map 阶段同一个 key 数据会发送一个 Reduce，当一个 key 数据过大就会倾斜。

并不是所有聚合操作都需要在 Reduce 段完成，很多聚合操作可以先在 Map 端进行部分聚合，最后在 Recuce 端得出结果

##### 开启Map端聚合参数

1. 是否在 Map 端进行聚合，默认为 true

```sql
set hive.map.aggr = true
```

2. 在 Map 端进行聚合操作的条目数目（可以设置更大）

```sql
set hive.groupby.mapaggr.checkinterval = 100000
```

3. 有数据倾斜时进行负载均衡，默认 false

```sql
set hive.groupby.skewindata = true
```

当选项为 true时，生成的查询计划会有两个 MR Job



如果数据量小，通过配置 `set hive.groupby.skewindata = true` 反而会降低效率，但是在大数据量且数据倾斜严重的情况下，效率还是很高的。



### Vectorization矢量

一次处理更多记录达到更高效率 （两个默认都是 true）

```sql
set hive.vectorized.execution.enable = true
set hive.vectorized.execution.reduce.enabled = true
```

### 多重模式

一堆SQL，多次对同表扫描

```sql
insert ... select .... from stu_buck where id = 10;
insert ... select .... from stu_buck where id = 11;
insert ... select .... from stu_buck where id = 12;
```

修改为扫描一次

```sql
from stu_buck
insert into stu_buck_once partition(id=10) select xxxx where id = 10
insert into stu_buck_once partition(id=11) select xxxx where id = 11
```

好像只可以用于分区、分桶模式



### in/exists 语句

例如：

```sql
select xx,xx from a where a.id in (select xx form b);

select xx,xx from a where exists (select id from b where a.id = b.id)
```

in 与 exists 区别

+ in 把数据放到内存中，一条一条对比
+ exists 则是把一条一条数据拿去匹配（推荐）



使用 join

```sql
select xx,xx from a join b on a.id = b.id
```

应该转换成 `left semi join` 实现（更推挤）

```sql
select xx,xx from a left semi join b on a.id = b.id;
```





## 多表查询优化



### CBO优化

类似于 MySQL 的查询优化器，选择最优方法去做查询操作

```sql
-- 默认开启
set hive.cbo.enable=ture;
set hive.compute.query.using.stats=ture;
set hive.stats.fetch.column.stats=ture;
set hive.stats.fetch.partition.stats=ture;
```



### 谓词下推

默认 true，往往用于关联查询。

```sql
set hive.optimize.ppd = true
```

先做条件过滤，再做关联查询

如果关掉 谓词下推，还需要关闭 CBO 优化，然后再用 Explain 查看执行计划。此时会先 join 操作然后再做条件查询。



### MapJoin

将小表数据放到 Map 内存中，避免 Reducer 操作

默认开启

```sql
set hive.auto.convert.join = true
```

阈值设置，默认 25M 以下认为是小表

```sql
set hive.mapjoin.smalltable.filesize= 25000000;
```



同过测试不同的大表 join 小表，查看 Explapin 执行计划，根据关键字查看 join是在map阶段还是 reduce 阶段，

+  Map Join Operator
+  Join Operator

注意： left join 就是会引发 map join 失效



### SMB Join

大表和大表进行 Join，使用分桶，两表join需要使用倍数关系的桶数量。





### 笛卡尔积

join 时，不加 on 条件，或者无效 on 条件。当 Hive 设定了为严格模式，不允许 HQL 语句出现笛卡尔积。

```sql
set hive.mapred.mode = strict;
```



