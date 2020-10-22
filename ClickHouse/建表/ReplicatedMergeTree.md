##### 语法：

```sql
CREATE TABLE table_name_local on cluster {cluster_name}
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

##### 示例1:

```sql
CREATE TABLE IF NOT EXISTS db.table_name_local on cluster cluster_2replicas
(
    EventDate DateTime, -- 时间字段
    CounterID UInt32,
    UserID UInt32,
    ver UInt16
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/db.table_name', '{replica}') 
PARTITION BY toYYYYMMDD(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID)) 
-- 支持 SECOND MINUTE HOUR DAY WEEK MONTH QUARTER YEAR
TTL EventDate + INTERVAL 1 MONTH 
SETTINGS 
    index_granularity = 8192, -- 索引粒度
    merge_with_ttl_timeout=86400,  --单位：秒
    ttl_only_drop_parts=1 
```

##### 示例2：

```sql
-- 创建本地表
CREATE TABLE test.test_replica_local on cluster cluster01_2replicas
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32,
    ver UInt16
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/test_replica', '{replica}') 
PARTITION BY toYYYYMM(EventDate) -- 分区字段
ORDER BY (CounterID, EventDate, intHash32(UserID))  -- 主键
SETTINGS index_granularity = 8192 --索引粒度

-- 创建分布式表
CREATE TABLE test.test_replica_all ON CLUSTER cluster02_no_replicas 
(
  `EventDate` DateTime, 
  `CounterID` UInt32, 
  `UserID` UInt32, 
  `ver` UInt16
) 
ENGINE = Distributed('cluster01_2replicas', 'test', 'test_replica_local', rand());

-- 基于本地表创建分布式表
CREATE TABLE test.test_replica_all ON CLUSTER cluster02_no_replicas 
AS test.test_replica_local
ENGINE = Distributed(cluster01_2replicas, test, test_replica_local, rand());

-- 写入数据
insert into test.test_replica_local (EventDate,CounterID,UserID,ver) values ('2011-10-01',10,1001,2001),('2011-10-01',10,1001,2001);

-- 查询
select count() from test.test_replica_local;
select count() from test.test_replica_all;
```

