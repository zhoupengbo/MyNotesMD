#### 生成 clickhouse.repo 文件

```bash
echo '[clickhouse-lts]
name=ClickHouse - LTS Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/clickhouse/rpm/lts/$basearch/
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/clickhouse/CLICKHOUSE-KEY.GPG
gpgcheck=1
enabled=1
EOF
' > /etc/yum.repos.d/clickhouse.repo
```

#### 重建 yum 缓存

```bash
yum clean all
yum makecache fast
```

#### 安装 clickhouse-server 和 clickhouse-client

```bash
yum install clickhouse-server clickhouse-client
```

