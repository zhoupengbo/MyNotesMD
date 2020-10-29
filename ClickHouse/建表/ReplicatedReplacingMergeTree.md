#### 1. 创建本地表

示例一：非分区表

```sql
CREATE TABLE IF NOT EXISTS db.table_name_local on cluster cluster_2replicas
(
    EventDate DateTime, -- 时间字段
    CounterID UInt32,
    UserID UInt32,
    _ver DateTime MATERIALIZED now() -- 版本号
) ENGINE = ReplicatedReplacingMergeTree('/clickhouse/tables/{layer}-{shard}/db.table_name', '{replica}', _ver) 
ORDER BY (CounterID, EventDate, intHash32(UserID)) -- 联合主键
SETTINGS index_granularity = 8192
```

示例二：非分区表+TTL（必须有时间列，指定合并周期）

```sql
CREATE TABLE IF NOT EXISTS db.table_name_local on cluster cluster_2replicas
(
    EventDate DateTime, -- 时间字段
    CounterID UInt32,
    UserID UInt32,
    _ver DateTime MATERIALIZED now() -- 版本号
) ENGINE = ReplicatedReplacingMergeTree('/clickhouse/tables/{layer}-{shard}/db.table_name', '{replica}') 
ORDER BY (CounterID, EventDate, intHash32(UserID)) 
-- 支持 SECOND MINUTE HOUR DAY WEEK MONTH QUARTER YEAR
TTL EventDate + INTERVAL 1 MONTH 
SETTINGS 
    index_granularity = 8192, -- 索引粒度
    merge_with_ttl_timeout=86400  --单位：秒
```

示例三：非分区表+TTL（必须有时间列，指定part删除方式）

```sql
CREATE TABLE IF NOT EXISTS db.table_name_local on cluster cluster_2replicas
(
    EventDate DateTime, -- 时间字段
    CounterID UInt32,
    UserID UInt32,
    _ver DateTime MATERIALIZED now() -- 版本号
) ENGINE = ReplicatedReplacingMergeTree('/clickhouse/tables/{layer}-{shard}/db.table_name', '{replica}') 
ORDER BY (CounterID, EventDate, intHash32(UserID)) 
-- 支持 SECOND MINUTE HOUR DAY WEEK MONTH QUARTER YEAR
TTL EventDate + INTERVAL 1 MONTH 
SETTINGS 
    index_granularity = 8192, -- 索引粒度
    ttl_only_drop_parts=1 
```

示例四：分区表

```sql
CREATE TABLE IF NOT EXISTS db.table_name_local on cluster cluster_2replicas
(
    EventDate DateTime, -- 时间字段
    CounterID UInt32,
    UserID UInt32,
    _ver DateTime MATERIALIZED now() -- 版本号
) ENGINE = ReplicatedReplacingMergeTree('/clickhouse/tables/{layer}-{shard}/db.table_name', '{replica}') 
PARTITION BY toYYYYMMDD(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID)) 
SETTINGS index_granularity = 8192
```

示例五：分区表+TTL（指定合并周期）

```sql
CREATE TABLE IF NOT EXISTS db.table_name_local on cluster cluster_2replicas
(
    EventDate DateTime, -- 时间字段
    CounterID UInt32,
    UserID UInt32,
    _ver DateTime MATERIALIZED now() -- 版本号
) ENGINE = ReplicatedReplacingMergeTree('/clickhouse/tables/{layer}-{shard}/db.table_name', '{replica}') 
PARTITION BY toYYYYMMDD(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID)) 
SETTINGS index_granularity = 8192,merge_with_ttl_timeout=86400  --单位：秒
```

示例六：分区表+TTL（指定part删除方式）

```sql
CREATE TABLE IF NOT EXISTS db.table_name_local on cluster cluster_2replicas
(
    EventDate DateTime, -- 时间字段
    CounterID UInt32,
    UserID UInt32,
    _ver DateTime MATERIALIZED now() -- 版本号
) ENGINE = ReplicatedReplacingMergeTree('/clickhouse/tables/{layer}-{shard}/db.table_name', '{replica}') 
PARTITION BY toYYYYMMDD(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID)) 
SETTINGS index_granularity = 8192,ttl_only_drop_parts=1 
```

#### 2. 创建视图

示例一：

```sql
CREATE VIEW IF NOT EXISTS db.table_name_view on cluster cluster_2replicas AS
SELECT
  user_id ,
  argMax(score, create_time) AS score, 
  argMax(deleted, create_time) AS deleted,
  max(create_time) AS ctime 
FROM test_rmt_local 
GROUP BY user_id
HAVING deleted = 0;
```

