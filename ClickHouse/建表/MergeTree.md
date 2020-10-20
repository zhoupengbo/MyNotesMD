语法：

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
ORDER BY expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...]
```

示例1：

```sql
CREATE TABLE test.table_name on cluster {cluster_name} (
    `id` Int32, 
    `execution_time` Date, 
    `keeper_code` String
) 
ENGINE = MergeTree() 
PARTITION BY toYYYYMMDD(execution_time) -- 分区字段
ORDER BY id -- 主键
```

##### 示例2:

```sql
-- 单节点创建
CREATE TABLE test.test_local
(
    `FlightDate` Date,
    `Year` UInt16
)
ENGINE = MergeTree()
ORDER BY (FlightDate, Year);

-- 分布式创建
CREATE TABLE test.test_local ON cluster cluster02_no_replicas 
(
    `FlightDate` Date,
    `Year` UInt16
)
ENGINE = MergeTree()
ORDER BY (FlightDate, Year);

-- 创建分布式表
CREATE TABLE test.test_all ON cluster cluster02_no_replicas 
AS test.test_local 
ENGINE = Distributed(cluster02_no_replicas, test, test_local, rand());

-- 写入数据
insert into test.test_local (FlightDate,Year)values('2003-10-12',2003);
insert into test.test_local (FlightDate,Year)values('2003-10-12',2003);
insert into test.test_all (FlightDate,Year)values('2003-10-12',2003);
insert into test.test_all (FlightDate,Year)values('2004-10-12',2004);

-- 查询
select * from  test.test_local;
select * from  test.test_all;
```

