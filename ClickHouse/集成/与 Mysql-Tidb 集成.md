##### 1. 创建Mysql/Tidb表

```sql
CREATE TABLE `test` (
  `unique_id` int(11) NOT NULL AUTO_INCREMENT,
  `id` bigint(20) DEFAULT NULL,
  `name` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`unique_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin AUTO_INCREMENT=30002;
```

##### 2. 创建ClickHouse表

```
# 测试
CREATE TABLE test.test_mysql_table ( \
    `unique_id` Int8, \
    `id` Int16, \
    `name` String \
) ENGINE = MySQL('xx.xx.xx.xx:port', 'test', 'test', 'bi_dev_user', 'ziroomdb');
 
 
insert into test_mysql_table VALUES (3,300,'li');
```

###### 参考：

- https://clickhouse.tech/docs/en/engines/table-engines/integrations/mysql/

