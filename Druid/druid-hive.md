#### 建表：

```sql
SET hive.druid.broker.address.default=10.10.20.30:8082;

CREATE TABLE ssb_druid_hive
STORED BY 'org.apache.hadoop.hive.
druid.DruidStorageHandler'
TBLPROPERTIES (
"druid.segment.granularity" = "MONTH",
"druid.query.granularity" = "DAY")
AS
SELECT
	cast(time as timestamp) as `__time`, 
	cast(c_city as string) c_city, 
	cast(c_nation as string) c_nation
FROM T WHERE...
```

