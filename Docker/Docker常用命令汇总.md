#### 			1. 搜索镜像

```powershell
docker search `关键词`
```

#### 2. 拉取镜像

```powershell
docker pull `镜像名`
```

#### 3. 启动容器

```shell
docker run -d -p xxxx
docker run -it -d -p 8888:80 --name="nginx-test" nginx
```

#### 4. 查看容器

```powershell
docker ps | grep xxxx
```

#### 5. 进入容器

```shell
docker exec -it `容器id` sh
```

#### 6. 拷贝镜像内容到本地

```shell
docker cp `容器id`:`文件路径` `本地路径`
```

#### 7. 停止容器

```shell
docker stop `容器id`
```

#### 8. 删除镜像

```shell
docker rmi `镜像ID
```

#### 9. 删除容器

```shell
docker rm `容器id`
```

