只需要在提供查询入口的机器上创建，并不一定在所有机器上创建。

```sql
CREATE TABLE table_name_all on cluster {cluster_name}
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32,
    ver UInt16
) Distributed('集群名称', '库名', '表名', '数据分布算法')
```

示例：

```sql
CREATE TABLE table_name_all on cluster perftest_3shards_2replicas
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32,
    ver UInt16
) Distributed('perftest_3shards_2replicas', 'test', 'table_name', rand())
```

