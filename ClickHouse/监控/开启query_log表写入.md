开启query_thread_log表同query_log表。

#### 第一步：

在/etc/clickhouse-server/config.xml添加以下配置：

```xml
<part_log>
  <database>system</database>
  <table>part_log</table>
  <flush_interval_milliseconds>7500</flush_interval_milliseconds>
</part_log>
<query_thread_log>
  <database>system</database>
  <table>query_thread_log</table>
  <partition_by>toYYYYMM(event_date)</partition_by>
  <flush_interval_milliseconds>7500</flush_interval_milliseconds>
</query_thread_log>
```

#### 第二步：

##### 1. 指定user配置

在/etc/clickhouse-server/users.xml中profile模块添加以下配置：

```xml
<readonly>
	<readonly>1</readonly>
  <log_queries>1</log_queries>
  <log_query_threads>1</log_query_threads>
</readonly>
```

##### 2. 指定query配置

在执行查询时URL中添加param。

```
/?log_queries=1&log_query_threads=1
```

#### 第三步：

##### 1. 创建分布式表

```sql
CREATE TABLE IF NOT EXISTS system.query_log_all
ON CLUSTER cluster_2replicas
AS system.query_log
ENGINE = Distributed(cluster_2replicas,system,query_log,rand());
```

##### 2. 设置生命周期

```sql
ALTER TABLE system.query_log ON CLUSTER all_nodes MODIFY TTL event_date + INTERVAL 365 DAY;
ALTER TABLE system.query_thread_log ON CLUSTER all_nodes MODIFY TTL event_date + INTERVAL 30 DAY;
ALTER TABLE system.trace_log ON CLUSTER all_nodes MODIFY TTL event_date + INTERVAL 30 DAY;
ALTER TABLE system.metric_log ON CLUSTER all_nodes MODIFY TTL event_date + INTERVAL 365 DAY;
```

