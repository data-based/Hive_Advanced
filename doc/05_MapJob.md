## HiveMap优化



#### 并行执行

可以是 Hive执行的任何阶段，注意，需要有较高的资源才可以开启，避免资源争抢，集群优势教导

```sql
set hive.exec.parallel = true;

set hive.exec.parallel.thread.number=16;
```

