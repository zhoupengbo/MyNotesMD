#### 1. 单机版安装

###### 环境检查：检查当前CPU是否支持SSE 4.2

```shell
grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"
```

###### 1.1 创建用户

```shell
groupadd clickhouse
useradd clickhouse -r -m -g clickhouse
passwd clickhouse
```

###### 1.2 rpm安装

```shell
rpm -ivh clickhouse-common-static-19.17.9.60-2.x86_64.rpm
rpm -ivh clickhouse-server-19.17.9.60-2.noarch.rpm
rpm -ivh clickhouse-client-19.17.9.60-2.noarch.rpm
```

###### 1.3 相关配置文件

- /etc/security/limits.d/clickhouse.conf  打开文件数设置
- /etc/clickhouse-server/config.xml  端口配置、本地机器名配置、内存设置等
- /etc/clickhouse-server/users.xml   权限、配额设置
- metrika.xml 集群配置、ZK配置、分片配置等

###### 1.4 修改相关配置，自定义数据目录

```shell
mkdir /mysqldata/clickhouse
# 修改以下目录
vi /etc/clickhouse-server/config.xml
 
<path>/mysqldata/clickhouse/</path>
<tmp_path>/mysqldata/clickhouse/tmp/</tmp_path>
<user_files_path>/mysqldata/clickhouse/user_files/</user_files_path>
<format_schema_path>/mysqldata/clickhouse/format_schemas/</format_schema_path>
```

###### 1.5 启动

```shell
sudo service clickhouse-server start
```

###### 1.6 进入命令行

```shell
clickhouse-client  // 默认情况下它使用’default’用户无密码的与localhost:9000服务建立连接。
clickhouse-client --host=example.com  // 连接远程服务
```

#### 2. 集群模式安装

在每台机器上按单机模式先装好clickhouse。

###### 2.1 配置文件拷贝

```shell
# 在 /etc/clickhouse-server/config.xml 中添加以下配置：
vim /etc/clickhouse-server/config.xml
    <include_from>/etc/clickhouse-server/metrika.xml</include_from>
    <listen_host>0.0.0.0</listen_host>
 
scp /etc/clickhouse-server/config.xml root@xx.xxx.xxx.xx:/etc/clickhouse-server/
```

发送到其他服务器。需要修改的地方如下：分别改为主机名

```xml
<macros>
    <replica>xxx</replica>
</macros>
```

###### 2.2 新建 /etc/clickhouse-server/metrika.xml 文件

添加以下内容：

```xml
<yandex>   
    <clickhouse_remote_servers>
        <perftest_3shards_2replicas>
            <shard>
                <replica>
                    <host>drd.es1.xxxx.com</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <replica>
                    <host>drd.es2.xxxx.com</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <replica>
                    <host>drd.es3.xxxx.com</host>
                    <port>9000</port>
                </replica>
            </shard>
        </perftest_3shards_2replicas>
    </clickhouse_remote_servers>
     
    <macros>
        <replica>ch1</replica>
    </macros>
     
    <!-- 监听网络 -->
    <networks>
        <ip>::/0</ip>
    </networks>
     
    <!-- ZK 临时搭了一个本地zk -->
    <zookeeper-servers>
        <node index="1">
            <host>drd.es1.xxxx.com</host>
            <port>2181</port>
        </node>
    </zookeeper-servers>
     
    <!-- 数据压缩算法  -->
    <clickhouse_compression>
        <case>
            <min_part_size>10000000000</min_part_size>
            <min_part_size_ratio>0.01</min_part_size_ratio>
            <method>lz4</method>
        </case>
    </clickhouse_compression>
</yandex>
```

###### **2.3 启动各个节点**

```shell
sudo service clickhouse-server start
```

###### 2.4 验证集群安装

```sql
select * from system.clusters;
```

###### 2.5 测试

```sql
create database test ON cluster perftest_3shards_2replicas;
USE test;

CREATE TABLE test.ontime_local ON cluster perftest_3shards_2replicas (FlightDate Date,Year UInt16) ENGINE = MergeTree(FlightDate, (Year, FlightDate), 8192);

CREATE TABLE test.ontime_all ON cluster perftest_3shards_2replicas AS test.ontime_local ENGINE = Distributed(perftest_3shards_2replicas, test, ontime_local, rand());
 
insert into test.ontime_all (FlightDate,Year)values('2001-10-12',2001);
insert into test.ontime_all (FlightDate,Year)values('2002-10-12',2002);
insert into test.ontime_all (FlightDate,Year)values('2003-10-12',2003);
insert into test.ontime_all (FlightDate,Year)values('2004-10-12',2004);
 
select * from  test.ontime_all;
```

#### 2.6 停止服务

```shell
sudo service clickhouse-server stop
```



#### 参考：

- https://clickhouse.tech/docs/en/getting-started/install/
- http://www.clickhouse.com.cn/topic/5a366e97828d76d75ab5d5a0
- https://opensource.actionsky.com/20200827-clickhouse/
- https://www.cnblogs.com/jiashengmei/p/11991243.html
- https://repo.yandex.ru/clickhouse/rpm/stable/x86_64/  （下载地址）