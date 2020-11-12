# **Docker技术分享**



### **一：安装Docker** 



### **二：关于容器技术**

#### chroot

什么是 chroot 呢？下面是 chroot 维基百科定义：

```
chroot 是在 Unix 和 Linux 系统的一个操作，针对正在运作的软件行程和它的子进程，改变它外显的根目录。一个运行在这个环境下，经由 chroot 设置根目录的程序，它不能够对这个指定根目录之外的文件进行访问动作，不能读取，也不能更改它的内容。
```

创建一个rootfs文件 mkdir rootfs

```
$ cd rootfs 
$ docker export $(docker create busybox) -o busybox.tar
$ tar -xf busybox.tar

```

执行完上面的命令后，在 rootfs 目录下，我们会得到一些目录和文件。下面我们使用 ls 命令查看一下 rootfs 目录下的内容。

```
$ ls
bin  busybox.tar  dev  etc  home  proc  root  sys  tmp  usr  var
```

```
$ chroot /home/centos/rootfs /bin/sh

```

此时，们的命令行窗口已经处于上述命令启动的 sh 进程中。

```
/ # /bin/ls /

bin  busybox.tar  dev  etc  home  proc  root  sys  tmp  usr  var

```

这里可以看到当前进程的根目录已经变成了主机上的 /home/centos/rootfs 目录。这样就实现了当前进程与主机的隔离。到此为止，一个目录隔离的容器就完成了

```
# /bin/ip route
```

#### Namespace资源隔离

Namespace 是 Linux 内核的一项功能，该功能对内核资源进行隔离，使得容器中的进程都可以在单独的命名空间中运行，并且只可以访问当前容器命名空间的资源。Namespace 可以隔离进程 ID、主机名、用户 ID、文件名、网络访问和进程间通信等相关资源。

Docker 主要用到以下五种命名空间。

- pid namespace：用于隔离进程 ID。
- net namespace：隔离网络接口，在虚拟的 net namespace 内用户可以拥有自己独立的 IP、路由、端口等。
- mnt namespace：文件系统挂载点隔离。
- ipc namespace：信号量,消息队列和共享内存的隔离。
- uts namespace：主机名和域名的隔离。

#### Cgroups资源限制

Cgroups 是一种 Linux 内核功能，可以限制和隔离进程的资源使用情况（CPU、内存、磁盘 I/O、网络等）。在容器的实现中，Cgroups 通常用来限制容器的 CPU 和内存等资源的使用。

#### 联合文件系统

联合文件系统，又叫 UnionFS，是一种通过创建文件层进程操作的文件系统，因此，联合文件系统非常轻快。Docker 使用联合文件系统为容器提供构建层，使得容器可以实现写时复制以及镜像的分层构建和存储。常用的联合文件系统有 AUFS、Overlay 和 Devicemapper 等。

各Linux版本的UnionFS不同 docker info 查看

```
Storage Driver: overlay2
```

**Docker镜像的设计中，引入了层（layer）的概念**，也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量rootfs（一个目录），这样应用A和应用B所在的容器共同引用相同的Debian操作系统层、Golang环境层（作为只读层），而各自有各自应用程序层，和可写层。启动容器的时候通过UnionFS把相关的层挂载到一个目录，作为容器的根文件系统。

需要注意的是，rootfs只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。这就意味着，如果你的应用程序需要配置内核参数、加载额外的内核模块，以及跟内核进行直接的交互，你就需要注意了：**这些操作和依赖的对象，都是宿主机操作系统的内核，它对于该机器上的所有容器来说是一个“全局变量”，牵一发而动全身。**

https://blog.csdn.net/songcf_faith/article/details/8278794



### **三 :镜像 容器 仓库**

#### 镜像

**镜像是 Docker 容器启动的先决条件。**

两种方式构建我们的镜像 

方法一：自己编写 DockeFile

方法二：docker pull 拉取

#### 容器

容器是镜像的运行实体。镜像是静态的只读文件，而容器带有运行时需要的可写文件层，并且容器中的进程属于运行状态。即**容器运行着真正的应用进程。容器有初建、运行、停止、暂停和删除五种状态。**

#### 仓库

Docker 的镜像仓库类似于代码仓库，用来存储和分发 Docker 镜像。镜像仓库分为公共镜像仓库和私有镜像仓库。

#### 	cs 架构

- `runC`是 Docker 官方按照 OCI 容器运行时标准的一个实现。通俗地讲，runC 是一个用来运行容器的轻量级工具，是真正用来运行容器的。
- `containerd`是 Docker 服务端的一个核心组件，它是从`dockerd`中剥离出来的 ，它的诞生完全遵循 OCI 标准，是容器标准化后的产物。`containerd`通过 containerd-shim 启动并管理 runC，可以说`containerd`真正管理了容器的生命周期。

https://opencontainers.org/



### **四 ：docker操作**

- 拉取镜像，使用`docker pull`命令拉取远程仓库的镜像到本地 ；
- 重命名镜像，使用`docker tag`命令“重命名”镜像 ；
- 查看镜像，使用`docker image ls`或`docker images`命令查看本地已经存在的镜像 ；
- 删除镜像，使用`docker rmi`命令删除无用镜像 ；
- 构建镜像，构建镜像有两种方式。第一种方式是使用`docker build`命令基于 Dockerfile 构建镜像，也是我比较推荐的镜像构建方式；第二种方式是使用`docker commit`命令基于已经运行的容器提交为镜像。

#### 拉取镜像

Docker 镜像的拉取使用`docker pull`命令， 命令格式一般为 docker pull [Registry]/[Repository]/[Image]:[Tag]。

- Registry 为注册服务器，Docker 默认会从 docker.io 拉取镜像，如果你有自己的镜像仓库，可以把 Registry 替换为自己的注册服务器。
- Repository 为镜像仓库，通常把一组相关联的镜像归为一个镜像仓库，`library`为 Docker 默认的镜像仓库。
- Image 为镜像名称。
- Tag 为镜像的标签，如果你不指定拉取镜像的标签，默认为`latest`。

实际上执行`docker pull busybox`命令，都是先从本地搜索，如果本地搜索不到`busybox`镜像则从 Docker Hub 下载镜像。

#### 查看镜像

Docker 镜像查看使用`docker images`或者`docker image ls`命令

`docker images`命令列出本地所有的镜像。

查询指定的镜像，可以使用`docker image ls`命令来查询

#### “重命名”镜像

如果你想要自定义镜像名称或者推送镜像到其他镜像仓库，你可以使用`docker tag`命令将镜像重命名。`docker tag`的命令格式为 docker tag [SOURCE_IMAGE][:TAG] [TARGET_IMAGE][:TAG]。

```
$ docker tag busybox:latest mybusybox:latest
```

#### 删除镜像

你可以使用`docker rmi`或者`docker image rm`命令删除镜像。

#### 构建镜像

构建镜像主要有两种方式：

1. 使用`docker commit`命令从运行中的容器提交为镜像；
2. 使用`docker build`命令从 Dockerfile 构建镜像。

先创建一个镜像

```
$ docker run --rm --name=busybox -it busybox bash
```

执行完上面的命令后，当前窗口会启动一个 busybox 容器并且进入容器中。在容器中，执行以下命令创建一个文件并写入内容：

```
/ # touch hello.txt && echo "I love Docker. " > hello.txt
```

此时在容器的根目录下，已经创建了一个 hello.txt 文件，并写入了 "I love Docker. "。下面，我们新打开另一个命令行窗口，运行以下命令提交镜像:

```
$ docker commit busybox busybox:hello

sha256:cbc6406aaef080d1dd3087d4ea1e6c6c9915ee0ee0f5dd9e0a90b03e2215e81c

```

然后使用上面讲到的`docker image ls`命令查看镜像：

```
$ docker image ls busybox
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             hello               cbc6406aaef0        2 minutes ago       1.22MB
busybox             latest              018c9d7b792b        4 weeks ago         1.22MB
```

第二种方式是最重要也是最常用的镜像构建方式：Dockerfile。Dockerfile 是一个包含了用户所有构建命令的文本。通过`docker build`命令可以从 Dockerfile 生成镜像。

使用 Dockerfile 构建镜像具有以下特性：

- Dockerfile 的每一行命令都会生成一个独立的镜像层，并且拥有唯一的 ID；
- Dockerfile 的命令是完全透明的，通过查看 Dockerfile 的内容，就可以知道镜像是如何一步步构建的；
- Dockerfile 是纯文本的，方便跟随代码一起存放在代码仓库并做版本管理。

看到使用 Dockerfile 的方式构建镜像有这么多好的特性，你是不是已经迫不及待想知道如何使用了。别着急，我们先学习下 Dockerfile 常用的指令。

| Dockerfile 指令 | 指令简介                                                     |
| --------------- | ------------------------------------------------------------ |
| FROM            | Dockerfile 除了注释第一行必须是 FROM ，FROM 后面跟镜像名称，代表我们要基于哪个基础镜像构建我们的容器。 |
| RUN             | RUN 后面跟一个具体的命令，类似于 Linux 命令行执行命令。      |
| ADD             | 拷贝本机文件或者远程文件到镜像内                             |
| COPY            | 拷贝本机文件到镜像内                                         |
| USER            | 指定容器启动的用户                                           |
| ENTRYPOINT      | 容器的启动命令                                               |
| CMD             | CMD 为 ENTRYPOINT 指令提供默认参数，也可以单独使用 CMD 指定容器启动参数 |
| ENV             | 指定容器运行时的环境变量，格式为 key=value                   |
| ARG             | 定义外部变量，构建镜像时可以使用 build-arg = 的格式传递参数用于构建 |
| EXPOSE          | 指定容器监听的端口，格式为 [port]/tcp 或者 [port]/udp        |
| WORKDIR         | 为 Dockerfile 中跟在其后的所有 RUN、CMD、ENTRYPOINT、COPY 和 ADD 命令设置工作目录。 |

demo

```
FROM centos:7
COPY nginx.repo /etc/yum.repos.d/nginx.repo
RUN yum install -y nginx
EXPOSE 80
ENV HOST=mynginx
CMD ["nginx","-g","daemon off;"]
```

- 第一行表示我要基于 centos:7 这个镜像来构建自定义镜像。这里需要注意，每个 Dockerfile 的第一行除了注释都必须以 FROM 开头。
- 第二行表示拷贝本地文件 nginx.repo 文件到容器内的 /etc/yum.repos.d 目录下。这里拷贝 nginx.repo 文件是为了添加 nginx 的安装源。
- 第三行表示在容器内运行`yum install -y nginx`命令，安装 nginx 服务到容器内，执行完第三行命令，容器内的 nginx 已经安装完成。
- 第四行声明容器内业务（nginx）使用 80 端口对外提供服务。
- 第五行定义容器启动时的环境变量 HOST=mynginx，容器启动后可以获取到环境变量 HOST 的值为 mynginx。
- 第六行定义容器的启动命令，命令格式为 json 数组。这里设置了容器的启动命令为 nginx ，并且添加了 nginx 的启动参数 -g 'daemon off;' ，使得 nginx 以前台的方式启动。

### **镜像的实现原理**

其实 Docker 镜像是由一系列镜像层（layer）组成的，每一层代表了镜像构建过程中的一次提交。下面以一个镜像构建的 Dockerfile 来说明镜像是如何分层的。

```
FROM busybox
COPY test /tmp/test
RUN mkdir /tmp/testdir
```

面的 Dockerfile 由三步组成：

第一行基于 busybox 创建一个镜像层；

第二行拷贝本机 test 文件到镜像内；

第三行在 /tmp 文件夹下创建一个目录 testdir。



### **五：容器的操作**

#### （1）创建并启动容器

docker  create 创建的容器需要docker start  启动 =  docker run

当使用`docker run`创建并启动容器时，Docker 后台执行的流程为：

- Docker 会检查本地是否存在 busybox 镜像，如果镜像不存在则从 Docker Hub 拉取 busybox 镜像；
- 使用 busybox 镜像创建并启动一个容器；
- 分配文件系统，并且在镜像只读层外创建一个读写层；
- 从 Docker IP 池中分配一个 IP 给容器；
- 执行用户的启动命令运行镜像。

docker run  -it  -rm  --name -p  

- -it 交互模式
- --rm  退出时删除容器
- --name 别名
- -p 端口
- -v 映射
- 

docker stop 终止容器 docker start 开启容器

docker restart为先stop 后start

docker  rm 删除容器

导出容器

docker export busybox > busybox.tar

导入容器

docker import busybox.tar busybox:test

六 ：Docker-compose 文件





