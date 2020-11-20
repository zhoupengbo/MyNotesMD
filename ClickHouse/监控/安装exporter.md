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

```
#! /bin/bash
# 启动命令
nohup xxxx &
pid=$(ps -ef | grep xxx | grep -v grep | awk '{print $2}')
if [ $pid ];then
    echo "status: running; pid: $pid" 
else 
    echo "status: not running!"
fi
```

##### status.sh

```
#! /bin/bash
pid=$(ps -ef | grep xxx | grep -v grep | awk '{print $2}')
if [ $pid ];then
    echo "status: running; pid: $pid" 
else 
    echo "status: stopped!"
fi
```

##### stop.sh

```
#!/bin/bash

# ps -ef|grep xxx |awk '{print $2}'|xargs kill -9
pid=$(ps -ef | grep xxx | grep -v grep | awk '{print $2}')
if [ $pid ];then 
    kill $pid
    echo "kill pid: $pid"
else
    echo "service is not running!"
fi
```

##### restart.sh

```
#!/bin/bash

# ps -ef|grep xxx |awk '{print $2}'|xargs kill -9
pid=$(ps -ef | grep xxx | grep -v grep | awk '{print $2}')
if [ $pid ];then 
    kill $pid
    echo "kill pid: $pid"
else
    echo "service is not running!"
fi
nohup xxxx &
pid=$(ps -ef | grep xxx | grep -v grep | awk '{print $2}')
if [ $pid ];then
    echo "status: running; pid: $pid" 
else 
    echo "status: not running!"
fi
```



#### 相关链接

[https://github.com/f1yegor/clickhouse_exporter](https://github.com/f1yegor/clickhouse_exporter/blob/master/Dockerfile)

[https://hub.docker.com/r/f1yegor/clickhouse-exporter](https://hub.docker.com/r/f1yegor/clickhouse-exporter)

