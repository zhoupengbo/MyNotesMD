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
```

