#### 场景一：ClickHouse 写入 HDFS（不支持ORC）

```mysql
CREATE TABLE default.test_hdfs_table (`name` String, `value` UInt32) ENGINE = HDFS('hdfs://xx.xx.xxx.xx:8020/tmp/test_ch', 'CSV')

# 只能写一次
insert into default.test_hdfs_table (name,value) values ("zhou",20);
select * from default.test_hdfs_table;
```

#### 场景二：ClickHouse 读取 HDFS（Parquet）

支持多种文件格式，参考：https://clickhouse.tech/docs/en/interfaces/formats/#formats

```mysql
# 创建Hive表
CREATE TABLE test_parquet ( id string) STORED AS Parquet;

insert into test_parquet select '123';
insert into test_parquet select '456';
 
# 直接读取，支持通配符查询
select * from hdfs('hdfs://xx.xx.xxx.xx:8020/warehouse/tablespace/managed/hive/test.db/test_parquet/delta_0000001_0000001_0000/000000_0','Parquet','id String')

select * from hdfs('hdfs://xx.xx.xxx.xx:8020/warehouse/tablespace/managed/hive/test.db/test_parquet/delta_000000{1..2}_000000{1..2}_0000/000000_0','Parquet','id String')

select * from hdfs('hdfs://xx.xx.xxx.xx:8020/warehouse/tablespace/managed/hive/test.db/test_parquet/*/000000_0','Parquet','id String')
 
# 创建映射表进行读取
CREATE TABLE test_hdfs_parquet (`id` String) ENGINE = HDFS('hdfs://xx.xx.xxx.xx:8020/warehouse/tablespace/managed/hive/test.db/test_parquet/*/*','Parquet');

select * from test_hdfs_parquet;
```

#### 场景三：ClickHouse 读取 HDFS（ORC），不支持ACID表

```sql
# 创建Hive表（非ACID表）
CREATE TABLE `test_orc`( `id` string,name String) STORED AS  orc tblproperties("transactional"="false");
 
# 直接读取，支持通配符查询(hdfs区分大小写)
select * from hdfs('hdfs://xx.xx.xxx.xx:8020/warehouse/tablespace/managed/hive/test.db/test_orc/*', 'ORC','id String,name String');

select * from hdfs('hdfs://xx.xx.xxx.xx:8020/warehouse/tablespace/managed/hive/test.db/test_orc/000000_0_copy_{1..2}', 'ORC','id String,name String');
 
# 创建映射表进行读取
CREATE TABLE default.test_hdfs_orc (`id` String, `name` String) ENGINE = HDFS('hdfs://10.30.238.11:8020/warehouse/tablespace/managed/hive/test.db/test_orc/*', 'ORC')

select * from test_hdfs_orc;
```

####  场景四：HDFS 数据导入

```mysql
# 方式一：
CREATE TABLE testx (`id` String) ENGINE = MergeTree() ORDER BY (id); -- 创建本地表

insert into testx select * from hdfs('hdfs://xx.xx.xxx.xx:8020/warehouse/tablespace/managed/hive/test.db/test_parquet/*/000000_0','Parquet','id String')
 
# 方式二：
hdfs dfs -cat hdfs://xx.xx.xxx.xx:8020/warehouse/tablespace/managed/hive/test.db/test_parquet/delta_0000001_0000001_0000/000000_0 | /usr/local/clickhouse/bin/clickhouse client --query="INSERT INTO testx FORMAT Parquet"
 
# 方式三：先建映射表，然后insert into
 
# 其他第三方工具导入
```

#### 高可用配置

###### 1. 源码编译安装

```shell
cp /etc/hadoop/conf/hdfs-site.xml /usr/local/clickhouse/etc/
export LIBHDFS3_CONF=/usr/local/clickhouse/etc/hdfs-client.xml
```

###### 2. RPM包安装

```shell
cp /etc/hadoop/conf/hdfs-site.xml /etc/clickhouse-server
vi /etc/init.d/clickhouse-server
	CLICKHOUSE_PROGRAM_ENV="export LIBHDFS3_CONF=/etc/clickhouse-server/hdfs-client.xml;"
	
# 或者
sed -i "s/CLICKHOUSE_PROGRAM_ENV=\"\"/CLICKHOUSE_PROGRAM_ENV=\"export LIBHDFS3_CONF=\/etc\/clickhouse-server\/hdfs-client.xml;\"/g" /etc/init.d/clickhouse-server
```

