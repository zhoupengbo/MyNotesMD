```sql
CREATE TABLE table_name on cluster {cluster_name}
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32,
    ver UInt16
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/table_name', '{replica}') -- zookeeper 路径 副本编号
PARTITION BY toYYYYMM(EventDate) -- 分区字段
ORDER BY (CounterID, EventDate, intHash32(UserID))  -- 主键
SETTINGS index_granularity = 8192 --索引粒度
```

