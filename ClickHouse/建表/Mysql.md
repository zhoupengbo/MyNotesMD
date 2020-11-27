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

