```bash
cat <<EOF | tee /etc/yum.repos.d/proxysql.repo
[proxysql_repo]
name= ProxySQL YUM repository
baseurl=https://repo.proxysql.com/ProxySQL/proxysql-2.0.x/centos/\$releasever
gpgcheck=1
gpgkey=https://repo.proxysql.com/ProxySQL/repo_pub_key
EOF


yum install proxysql OR yum install proxysql-version

proxysql --clickhouse-server

mysql -uadmin -padmin -h127.0.0.1 -P6032

INSERT INTO clickhouse_users VALUES ('clicku','clickp',1,100);

mysql -u clicku -pclickp -h 127.0.0.1 -P6090
```

参考：

- https://my.oschina.net/roockee/blog/3108396
- https://proxysql.com/documentation/getting-started/
- https://github.com/sysown/proxysql/wiki/ClickHouse-Support
- https://proxysql.com/Documentation/ClickHouse-Configuration/
- https://proxysql.com/blog/clickhouse-and-proxysql-queries-rewrite/

