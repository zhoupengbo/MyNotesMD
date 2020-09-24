#### 1. vim /etc/clickhouse-server/config.xml

```xml
<!--新增-->
<mysql_port>9004</mysql_port>
```

#### 2. vim /etc/clickhouse-server/user.xml

```xml
<!--设置用户密码-->
<password_double_sha1_hex>xxxxxx</password_double_sha1_hex>
```

#### 3. 客户端登录

```shell
mysql -h 10.30.238.17 -u clickhouse -P 9004 -p
```

