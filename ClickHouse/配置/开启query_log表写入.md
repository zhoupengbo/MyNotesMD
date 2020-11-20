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

