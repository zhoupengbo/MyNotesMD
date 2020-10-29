```mysql
-- system.parts
SELECT
	partition,name,modification_time,partition_id,database,table,engine,path 
FROM system.parts 
WHERE table = 'query_log'
ORDER BY modification_time desc
LIMIT 10;

-- system.clusters
SELECT * FROM  system.clusters;

-- 查看各个库表，占用的存储空间大小
SELECT
database,
table,
formatReadableSize ( sum( bytes ) ) AS size
FROM
system.parts
WHERE
active
GROUP BY database,table
ORDER BY sum( bytes ) DESC;

-- 查询物化视图
SELECT 
    partition, 
    name, 
    rows, 
    bytes_on_disk, 
    modification_time, 
    min_date, 
    max_date, 
    engine
FROM system.parts
WHERE (database = 'dw') AND (table = '.inner.main_site_minute_pv_uv')
```

