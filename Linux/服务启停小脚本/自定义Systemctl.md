#### 自定义：

##### Vim /etc/systemd/system/clickhouse-exporter.service

```
[Unit]
Description=ClickHouse Exporter (analytic DBMS for big data)
Requires=network-online.target
After=network-online.target

[Service]
Type=forking
User=root
Group=root
Restart=always
RestartSec=30
ExecStart=/home/data/chexporter/start.sh
ExecStop=/home/data/chexporter/stop.sh
LimitCORE=infinity
LimitNOFILE=500000

[Install]
WantedBy=multi-user.target
```

