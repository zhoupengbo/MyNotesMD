```bash
#!/bin/bash
# 拉取镜像
docker pull f1yegor/clickhouse-exporter
# 运行容器
docker run -d -p 9116:9116 f1yegor/clickhouse-exporter -scrape_uri=http://localhost:8123/
# 查看容器id
docker ps | grep clickhouse
# 进入容器
docker exec -it 860c234aaebf sh
# 从容器中拷贝可执行包
docker cp 860c234aaebf:/usr/local/bin/clickhouse_exporter ./
# 停止容器
docker stop 860c234aaebf

# 设置环境变量
export CLICKHOUSE_USER=default
export CLICKHOUSE_PASSWORD=xxxxx
# 启动服务
nohup ./clickhouse_exporter -scrape_uri=http://localhost:8123/ &
```

##### start.sh

```bash
#! /bin/bash
pid=$(ps -ef | grep clickhouse_exporter | grep -v grep | awk '{print $2}')
if [ $pid ];then
    echo "status: running; pid: $pid"
else
    echo "status: stopped!"
fi
```

##### status.sh

```bash
#! /bin/bash
pid=$(ps -ef | grep clickhouse_exporter | grep -v grep | awk '{print $2}')
if [ $pid ];then
    echo "status: running; pid: $pid"
else
    echo "status: stopped!"
fi
[root@ch05 chexporter]# cat status.sh
#! /bin/bash
pid=$(ps -ef | grep clickhouse_exporter | grep -v grep | awk '{print $2}')
if [ $pid ];then
    echo "status: running; pid: $pid"
else
    echo "status: stopped!"
fi
```

##### stop.sh

```bash
#!/bin/bash
# ps -ef | grep clickhouse_exporter | grep -v grep | awk '{print $2}' | xargs kill -9
pid=$(ps -ef | grep clickhouse_exporter | grep -v grep | awk '{print $2}')
if [ $pid ];then
    kill $pid
    echo "kill pid: $pid"
else
    echo "clickhouse_exporter is not running!"
fi
```

##### restart.sh

```bash
#!/bin/bash
# 停止
pid=$(ps -ef | grep clickhouse_exporter | grep -v grep | awk '{print $2}')
if [ $pid ];then
    kill $pid
    echo "kill pid: $pid"
else
    echo "clickhouse_exporter is not running!"
fi
# 启动
export CLICKHOUSE_USER=readonly
export CLICKHOUSE_PASSWORD=KDfTLNhK
nohup /home/data/chexporter/clickhouse_exporter -scrape_uri=http://localhost:8123/ &
pid=$(ps -ef | grep clickhouse_exporter | grep -v grep | awk '{print $2}')
if [ $pid ];then
    echo "status: running; pid: $pid"
else
    echo "status: not running!"
fi
```

#### 自定义systemctl

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

```shell
systemctl daemon-reload
systemctl status clickhouse-exporter.service
systemctl start clickhouse-exporter.service
systemctl stop clickhouse-exporter.service
systemctl restart clickhouse-exporter.service
```



#### 相关链接

[https://github.com/f1yegor/clickhouse_exporter](https://github.com/f1yegor/clickhouse_exporter/blob/master/Dockerfile)

[https://hub.docker.com/r/f1yegor/clickhouse-exporter](https://hub.docker.com/r/f1yegor/clickhouse-exporter)

