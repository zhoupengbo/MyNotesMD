#### 建表语句

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = Kafka()
SETTINGS
    kafka_broker_list = 'host:port',
    kafka_topic_list = 'topic1,topic2,...',
    kafka_group_name = 'group_name',
    kafka_format = 'data_format'[,]
    [kafka_row_delimiter = 'delimiter_symbol',]
    [kafka_schema = '',]
    [kafka_num_consumers = N,]
    [kafka_max_block_size = 0,]
    [kafka_skip_broken_messages = N,]
    [kafka_commit_every_batch = 0,]
    [kafka_thread_per_consumer = 0]
```

###### 必传参数：

- kafka_broker_list：逗号分隔的broker列表（如：localhost:9092）；
- kafka_topic_list：kafka topic 列表；
- kafka_group_name：Kafka 消费组名称 (`group1`)。如果不希望消息在集群中重复，请在每个分片中使用相同的组名；
- kafka_format：使用与 SQL 部分的 `FORMAT` 函数相同表示方法，例如 `JSONEachRow`。

###### 选填参数：

- `kafka_row_delimiter` - 每个消息体（记录）之间的分隔符。
- `kafka_schema` – 如果解析格式需要一个 schema 时，此参数必填。
- `kafka_num_consumers` – 单个表的消费者数量。默认值是：`1`，如果一个消费者的吞吐量不足，则指定更多的消费者。消费者的总数不应该超过 topic 中分区的数量，因为每个分区只能分配一个消费者。

###### 虚拟列

- _topic
- __offset_
- __partition_
- __key
- _timestamp

###### 测试示例1：

```sql
-- kafka 管道
CREATE TABLE default.kafka_queue_test
(
    opType String,
    dbTable String,
    time DateTime64
) ENGINE = Kafka()
SETTINGS
    kafka_broker_list = 'xxx:9092,xxx:9092,xxx:9092',
    kafka_topic_list = 'test_topic',
    kafka_group_name = 'test_20200125',
    kafka_format = 'JSONEachRow';
-- 数据表
CREATE TABLE kafka_data_test (
    opType String,
    dbTable String,
    time DateTime64
) ENGINE = MergeTree()
order by tuple();
-- 雾化视图
CREATE MATERIALIZED VIEW kafka_consumer_test TO kafka_data_test
AS select * from kafka_queue_test;
```

###### 测试示例2:

```sql
-- kafka 管道
CREATE TABLE kafka_queue_chtest
(
    id String,
    name String,
    time DateTime
) ENGINE = Kafka()
SETTINGS
    kafka_broker_list = 'xx.xx.xx.xx:9092',
    kafka_topic_list = 'chtest-20210127',
    kafka_group_name = 'test-0127',
    kafka_format = 'JSONEachRow',
    kafka_num_consumers = 1,
    kafka_skip_broken_messages = 1,
    kafka_commit_every_batch = 10;
-- 存储原始数据，添加_ver雾化列
CREATE TABLE kafka_data_chtest (
    id String,
    name String,
    time DateTime,
    _ver DateTime64 MATERIALIZED now()
) ENGINE = MergeTree()
order by tuple();
-- 创建雾化视图表1
CREATE MATERIALIZED VIEW kafka_consumer_chtest TO 
AS select * from kafka_queue_chtest;
-- 创建预聚合表
CREATE TABLE kafka_data_chtest_agg (
    id String,
    name AggregateFunction(argMax, String, DateTime64),
    time AggregateFunction(argMax, DateTime, DateTime64)
) ENGINE = AggregatingMergeTree()
order by tuple();
-- 创建雾化视图表2
CREATE MATERIALIZED VIEW kafka_mview_chtest TO kafka_data_chtest_agg
AS select id, argMaxState(name, _ver) name , argMaxState(time, _ver) time
from kafka_data_chtest group by id;
-- 创建视图
create view kafka_view_chtest as 
select id, argMaxMerge(name) name, argMaxMerge(time) time from kafka_data_chtest_agg group by id;
```

###### 测试示例3:

```sql
-- kafka 管道
CREATE TABLE direct_kafka_queue_chtest
(
    id String,
    name String,
    time DateTime
) ENGINE = Kafka()
SETTINGS
    kafka_broker_list = 'xx.xx.xx.xx:9092',
    kafka_topic_list = 'chtest-20210127',
    kafka_group_name = 'test-0127-3',
    kafka_format = 'JSONEachRow',
    kafka_num_consumers = 1,
    kafka_skip_broken_messages = 1,
    kafka_commit_every_batch = 10
;
-- 数据表（聚合表）
CREATE TABLE direct_kafka_data_chtest (
    id String,
    name AggregateFunction(argMax, String, DateTime),
    max_time DateTime
) ENGINE = AggregatingMergeTree()
order by tuple()
;
-- 雾化视图
CREATE MATERIALIZED VIEW direct_kafka_mview_chtest TO direct_kafka_data_chtest
AS select id, argMaxState(name, time) name, max(time) max_time
from direct_kafka_queue_chtest group by id;
-- 访问视图
create view direct_kafka_view_chtest as 
select id, argMaxMerge(name) name, max(max_time) time from direct_kafka_data_chtest group by id;
```

###### 演示示例4:





config.xml

```xml
 <stream_poll_timeout_ms>30000</stream_poll_timeout_ms>
   <kafka>
      <enable_auto_commit>true</enable_auto_commit>
      <auto_commit_interval_ms>1000</auto_commit_interval_ms>
      <max_poll_interval_ms>600000</max_poll_interval_ms>
   </kafka>
```

users.xml

```xml
      <!-- Default settings. -->
      <default>
         <max_threads>50</max_threads>
         <background_schedule_pool_size>50</background_schedule_pool_size>
         <kafka_max_wait_ms>20000</kafka_max_wait_ms>
      </default>
```

The assumption is that kafka errors (i.e. unstable clickhouse kafka consumers) cause duplicate entries (we know the transactions are not supported yet) This is why we introduced the following configuration, but we are not sure whether this helps or not.

```xml
   <enable_auto_commit>true</enable_auto_commit>
   <auto_commit_interval_ms>1000</auto_commit_interval_ms>
```