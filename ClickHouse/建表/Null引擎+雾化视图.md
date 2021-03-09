```sql
-- 创建Null类型原始表
CREATE TABLE null_test (
    opType String,
    dbTable String,
    time DateTime64
) ENGINE = Null();
-- 创建雾化视图
CREATE MATERIALIZED VIEW null_test_mview ENGINE = MergeTree() 
partition by toYYYYMMDD(time) order by tuple()
AS select * from null_test;
-- 写入数据
insert into null_test values ('insert','test','2020-01-02 00:00:00');
-- 查询数据
select * from null_test;
select * from null_test_mview;
```

