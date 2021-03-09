语法：

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
) ENGINE = MySQL('host:port', 'database', 'table', 'user', 'password'[, replace_query, 'on_duplicate_clause']);
```

示例：

```sql
CREATE TABLE IF NOT EXISTS test.test_mysql_table ( \
    `unique_id` Int8, \
    `id` Int16, \
    `name` String \
) ENGINE = MySQL('xx.xx.xx.xx:port', 'test', 'test', 'zhou', 'zhou123');
```

数据库映射：

```sql
CREATE DATABASE [IF NOT EXISTS] db_name [ON CLUSTER cluster]
ENGINE = MySQL('host:port', ['database' | database], 'user', 'password')
```

基于mysql表创建clickhouse表：

```sql
CREATE TABLE tb1
ENGINE = MergeTree
PARTITION BY toYYYYMM(pay_time)
ORDER BY pay_time AS
SELECT *
FROM mysql('127.0.0.1:3306', 'yayun', 'tb1', 'ch_repl', '123') 
limit 1
```

