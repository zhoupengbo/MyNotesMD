### 部署安装

```shell
# 拉取镜像 version=2.1.3
$ docker pull harisekhon/hbase:latest
# 创建网络
$ docker network create hbase-network
# 运行
$ docker run -d -h docker-hbase \
        -p 2181:2181 \
        -p 8080:8080 \
        -p 8085:8085 \
        -p 9090:9090 \
        -p 9000:9000 \
        -p 9095:9095 \
        -p 16000:16000 \
        -p 16010:16010 \
        -p 16201:16201 \
        -p 16301:16301 \
        -p 16020:16020\
        --name hbase \
        harisekhon/hbase
# 绑定域名
$ vim /etc/hosts
	127.0.0.1 docker-hbase 
# 访问web-ui
  http://docker-hbase:16010/master-status
# 查看容器
$ docker ps
# 找到容器id，进入容器
$ docker exec -it <container ID前缀> bash
# 进入HBase Shell
$ hbase shell
# 进入ZK
$ hbase zkcli
```

### 参考文档

[https://hub.docker.com/r/harisekhon/hbase](https://hub.docker.com/r/harisekhon/hbase)