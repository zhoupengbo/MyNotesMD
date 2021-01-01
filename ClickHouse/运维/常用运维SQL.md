#### 1. 当前连接数

```sql
:) SELECT * FROM system.metrics WHERE metric LIKE '%Connection';

SELECT *
FROM system.metrics
WHERE metric LIKE '%Connection'

┌─metric────────────────┬─value─┬─description─────────────────────────────────────────────────────────┐
│ TCPConnection         │     2 │ Number of connections to TCP server (clients with native interface) │
│ HTTPConnection        │     1 │ Number of connections to HTTP server                                │
│ InterserverConnection │     0 │ Number of connections from other replicas to fetch parts            │
└───────────────────────┴───────┴─────────────────────────────────────────────────────────────────────┘
```

#### 2. 当前正在执行的查询

```sql
:) SELECT query_id, user, address, query  FROM system.processes ORDER BY query_id;

SELECT 
    query_id, 
    user, 
    address, 
    query
FROM system.processes
ORDER BY query_id ASC

┌─query_id─────────────────────────────┬─user────┬─address────────────┬─query──────────────────────────── ┐
│ 203f1d0e-944e-472d-8d8f-bae548ff9899 │ default │ ::ffff:10.37.129.4 │ SELECT query_id, user, address... │
│ fb7fba85-b2a0-4271-87ff-22da97ae511b │ default │ ::ffff:10.37.129.4 │ INSERT INTO hits_v1 FORMAT TSV                          ──────────  -   -   -   -    -  - ─────┴─────────┴────────────────────┴───────────────   ─────────────────┘
```

#### 3. 终止查询

```sql
:) KILL QUERY WHERE query_id='ff695827-dbf5-45ad-9858-a853946ea140';
```

#### 4. 存储空间统计

```sql
SELECT name,path,formatReadableSize(free_space) AS free,formatReadableSize(total_space) AS total,formatReadableSize(keep_free_space) AS reserved FROM system.disks

SELECT 
    name, 
    path, 
    formatReadableSize(free_space) AS free, 
    formatReadableSize(total_space) AS total, 
    formatReadableSize(keep_free_space) AS reserved
FROM system.disks

┌─name──────┬─path──────────────┬─free──────┬─total─────┬─reserved─┐
│ default   │ /chbase/data/     │ 36.35 GiB │ 49.09 GiB │ 0.00 B   │
│ disk_cold │ /chbase/cloddata/ │ 35.35 GiB │ 48.09 GiB │ 1.00 GiB │
│ disk_hot1 │ /chbase/data/     │ 36.35 GiB │ 49.09 GiB │ 0.00 B   │
│ disk_hot2 │ /chbase/hotdata1/ │ 36.35 GiB │ 49.09 GiB │ 0.00 B   │
└───────────┴───────────────────┴───────────┴───────────┴──────────┘
```

#### 5. 各数据库占用空间统计

```sql
SELECT 
    database, 
    formatReadableSize(sum(bytes_on_disk)) AS on_disk
FROM system.parts
GROUP BY database

┌─database─┬─on_disk──┐
│ system   │ 1.59 MiB │
│ default  │ 3.60 GiB │
└──────────┴──────────┘
```

#### 6. 每个列字段占用空间统计

每个列字段的压缩大小、压缩比率以及该列的每行数据大小的占比。

```sql
SELECT 
    database, 
    table, 
    column, 
    any(type), 
    sum(column_data_compressed_bytes) AS compressed, 
    sum(column_data_uncompressed_bytes) AS uncompressed, 
    round(uncompressed / compressed, 2) AS ratio, 
    compressed / sum(rows) AS bpr, 
    sum(rows)
FROM system.parts_columns
WHERE active AND database != 'system'
GROUP BY 
    database, 
    table, 
    column
ORDER BY 
    database ASC, 
    table ASC, 
    column ASC
```

#### 7. 慢查询

```sql
SELECT 
    user, 
    client_hostname AS host, 
    client_name AS client, 
    formatDateTime(query_start_time, '%T') AS started, 
    query_duration_ms / 1000 AS sec, 
    round(memory_usage / 1048576) AS MEM_MB, 
    result_rows AS RES_CNT, 
    result_bytes / 1048576 AS RES_MB, 
    read_rows AS R_CNT, 
    round(read_bytes / 1048576) AS R_MB, 
    written_rows AS W_CNT, 
    round(written_bytes / 1048576) AS W_MB, 
    query
FROM system.query_log
WHERE type = 2
ORDER BY query_duration_ms DESC
LIMIT 10
```

#### 8. 副本预警监控

通过下面的 SQL 语句对副本进行预警监控，其中各个预警的变量可以根据自身情况调整。

```sql
SELECT database, table, is_leader, total_replicas, active_replicas 
  FROM system.replicas 
 WHERE is_readonly 
    OR is_session_expired 
    OR future_parts > 30 
    OR parts_to_check > 20 
    OR queue_size > 30 
    OR inserts_in_queue > 20 
    OR log_max_index - log_pointer > 20 
    OR total_replicas < 2 
    OR active_replicas < total_replicas
```

#### 9. 查看库表资源占用情况

```sql
select database, \
       table, \
       sum(rows) AS "总行数", \
       formatReadableSize(sum(data_uncompressed_bytes))  as "原始大小",    \
       formatReadableSize(sum(data_compressed_bytes)) AS "压缩大小",  \
       round((sum(data_compressed_bytes) / sum(data_uncompressed_bytes)) * 100.,2) AS "压缩率/%"  \
from system.parts \
group by database,table  \
order by database;
-- 查看表占用空间
SELECT database, 
			 table, 
			 partition, 
       part_name, 
       active, 
       bytes_on_disk 
FROM system.parts 
			 where database not in 'system' 
ORDER BY database, table, partition, name;
-- 查看库大小
SELECT database, 
			 sum(bytes_on_disk) as db_size  
FROM system.parts   
GROUP BY database 
order by db_size desc;
```

#### 10. 查看SQL执行计划

```shell
clickhouse-client --send_logs_level=trace <<< 'SELECT * FROM hits_v1' > /dev/null
```

#### 11. 获取活跃会话

```sql
SELECT query_id, user, address, elapsed, query FROM system.processes ORDER BY query_id;
-- 或者
SHOW PROCESSLIST;
```

#### 12. 获取 mutation 操作

CH 的 update 和 delete 操作，算 DDL 操作，CH 称为 mutation，而要确定 mutation 队列，可以使用下面的 SQL：

```sql
SELECT * FROM system.mutations;
```

#### 13. kill mutation 操作 

```sql
KILL MUTATION mutation_id = 'trx_id';
```

#### 14. 正在执行的 SQL 概要 

正在执行的查询总次数、正在发生的合并操作总次数

```sql
select * from system.metrics limit 5;
```

#### 15. 累积 SQL 概要 

查看服务运行过程总的查询次数、总的 select 次数

```sql
select * from system.events limit 5;
```

#### 16. 正在后台运行的概要信息 

查看当前分配的内存、执行队列中的任务数量等

```sql
select * from system.asynchronous_metrics limit 5;
```

#### 17. 检查复制是否异常

```sql
SELECT database, table, is_leader, total_replicas,active_replicas 
FROM system.replicas  
WHERE is_readonly  
			OR is_session_expired 
			OR future_parts > 20  
			OR parts_to_check > 10  
			OR queue_size > 20 
			OR inserts_in_queue > 10  
			OR log_max_index - log_pointer > 10  
			OR total_replicas < 2  
			OR active_replicas < total_replicas;
```

#### 18. SQL 基准测试 

ClickHouse 自带基准测试工具 clickhouse-benchmark，用法如下：

```shell
echo "select * from testcluster_shard_1.tc_shard_all;" |clickhouse-benchmark -i 5
```

-i 5 表示 SQL 执行 5 次，会显示 QPS、RPS 及各百分位的查询执行时间。

#### 19. 解析表名

```sql
select 
    case when extract(replaceAll(query,'`',''), 'into\\s+(\\w+\\.\\w+)')   != '' 
    		 then extract(replaceAll(query,'`',''), 'into\\s+(\\w+\\.\\w+)')
         when extract(replaceAll(query,'`',''), 'INTO\\s+(\\w+\\.\\w+)')   != '' 
    		 then extract(replaceAll(query,'`',''), 'INTO\\s+(\\w+\\.\\w+)')
         when extract(replaceAll(query,'`',''), 'from\\s+(\\w+\\.\\w+)')   != '' 
         then extract(replaceAll(query,'`',''), 'from\\s+(\\w+\\.\\w+)')
         when extract(replaceAll(query,'`',''), 'FROM\\s+(\\w+\\.\\w+)')   != '' 
         then extract(replaceAll(query,'`',''), 'FROM\\s+(\\w+\\.\\w+)')
         when extract(replaceAll(query,'`',''), 'exists\\s+(\\w+\\.\\w+)') != '' 
         then extract(replaceAll(query,'`',''), 'exists\\s+(\\w+\\.\\w+)')
         when extract(replaceAll(query,'`',''), 'EXISTS\\s+(\\w+\\.\\w+)') != '' 
         then extract(replaceAll(query,'`',''), 'EXISTS\\s+(\\w+\\.\\w+)')
         when extract(replaceAll(query,'`',''), 'table\\s+(\\w+\\.\\w+)')  != '' 
         then extract(replaceAll(query,'`',''), 'table\\s+(\\w+\\.\\w+)')
         when extract(replaceAll(query,'`',''), 'TABLE\\s+(\\w+\\.\\w+)')  != '' 
         then extract(replaceAll(query,'`',''), 'TABLE\\s+(\\w+\\.\\w+)')
         else '无法解析' end as table_name, 
from query_log where query_id = '163FA05B3E482568'
```



