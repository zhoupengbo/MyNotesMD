示例一：未分区

```sql
-- 创建非分区本地表
CREATE TABLE test_rmt_local(
  user_id UInt64,
  score String,
  deleted UInt8 DEFAULT 0,
  create_time DateTime
)ENGINE= ReplacingMergeTree(create_time)
ORDER BY user_id;
-- 写入数据
INSERT INTO TABLE test_rmt_local(user_id,score)
WITH(
  SELECT ['A','B','C','D','E','F','G']
)AS dict
SELECT number AS user_id, dict[number%7+1] FROM numbers(10);
-- 写入数据
INSERT INTO TABLE test_rmt_local(user_id,score,create_time)
WITH(
  SELECT ['AA','BB','CC','DD','EE','FF','GG']
)AS dict
SELECT number AS user_id, dict[number%7+1], now() AS create_time FROM numbers(5);
-- 创建视图
CREATE VIEW test_rmt_view AS
SELECT
  user_id ,
  argMax(score, create_time) AS score, 
  argMax(deleted, create_time) AS deleted,
  max(create_time) AS ctime 
FROM test_rmt_local 
GROUP BY user_id
HAVING deleted = 0;
-- 查询视图
select * from test_rmt_view;
-- 修改数据
INSERT INTO TABLE test_rmt_local(user_id,score,create_time) VALUES(0,'AAAA',now());
-- 模拟删除，以增代删
INSERT INTO TABLE test_rmt_local(user_id,score,deleted,create_time) VALUES(0,'AAAA',1,now());
-- 基于视图创建分布式表
CREATE TABLE test_rmt_all on cluster perftest_2shards_2replicas
AS test_rmt_view
ENGINE = Distributed('perftest_2shards_2replicas', 'default', 'test_rmt_view', rand());
```

示例二：分区

```sql
-- 创建非分区本地表
CREATE TABLE test_rmtp_local(
  user_id UInt64,
  score String,
  deleted UInt8,
  create_time DateTime
)ENGINE= ReplacingMergeTree(create_time)
PARTITION BY toYYYYMMDD(create_time) 
ORDER BY user_id;
-- 写入数据
INSERT INTO TABLE test_rmtp_local(user_id,score)
WITH(
  SELECT ['A','B','C','D','E','F','G']
)AS dict
SELECT number AS user_id, dict[number%7+1] FROM numbers(5);
-- 写入数据
INSERT INTO TABLE test_rmtp_local(user_id,score,create_time)
WITH(
  SELECT ['AA','BB','CC','DD','EE','FF','GG']
)AS dict
SELECT number AS user_id, dict[number%7+1], now() AS create_time FROM numbers(5);
-- 写入数据
INSERT INTO TABLE test_rmtp_local(user_id,score,create_time)
WITH(
  SELECT ['AAA','BBB','CCC','DDD','EEE','FFF','GGG']
)AS dict
SELECT number AS user_id, dict[number%7+1], '2020-10-25 15:27:42' AS create_time FROM numbers(5);
-- 创建视图
CREATE VIEW test_rmtp_view AS
SELECT
  user_id ,toYYYYMMDD(create_time) as day,
  argMax(score, create_time) AS score, 
  argMax(deleted, create_time) AS deleted,
  max(create_time) AS ctime 
FROM test_rmtp_local 
GROUP BY user_id,toYYYYMMDD(create_time) 
HAVING deleted = 0;
-- 查询视图
select * from test_rmtp_view where day = 20201025;
-- 修改数据
INSERT INTO TABLE test_rmtp_local(user_id,score,create_time) VALUES(0,'AAAAA',now());
-- 模拟删除，以增代删
INSERT INTO TABLE test_rmtp_local(user_id,score,deleted,create_time) VALUES(0,'AAAA',1,now());
-- 基于视图创建分布式表
CREATE TABLE test_rmtp_all on cluster perftest_2shards_2replicas
AS test_rmtp_view
ENGINE = Distributed('perftest_2shards_2replicas', 'default', 'test_rmtp_view', rand());
```

特点：支持更新，删除，分区，定期覆盖