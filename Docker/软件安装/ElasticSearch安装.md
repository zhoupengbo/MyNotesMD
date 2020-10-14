### 单节点安装

```shell
# 拉取镜像
$ docker pull elasticsearch:7.8.1
# 创建网络
$ docker network create esnetwork
# 运行
$ docker run -d \
					 --name elasticsearch \
  					 --net esnetwork \
  					 -p 9200:9200 \
  					 -p 9300:9300 \
  					 -e "discovery.type=single-node" \
  					 elasticsearch:7.8.1
# 查看容器
$ docker ps
# 找到容器id，进入容器
$ docker exec -it <container ID前缀> bash
# Head插件访问
  http://localhost:9200/
```

### 参考链接

[https://hub.docker.com/_/elasticsearch?tab=description](https://hub.docker.com/_/elasticsearch?tab=description)



