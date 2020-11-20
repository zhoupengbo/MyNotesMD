#### 创建clickhouse用户(当前用户：root)

```bash
# 创建新用户
groupadd clickhouse
useradd clickhouse -r -m -g clickhouse
passwd clickhouse/clickhouse
```

#### 安装依赖项

```shell
yum install git cmake libicu-devel clang libicu-devel readline-devel mysql-devel openssl-devel unixODBC_devel bzip2 -y
```

#### 下载核心包并解压

```bash
cd /home/clickhouse
# 下载编译好的包
wget http://xx.xx.xxx.xx:port/clickhouse-19.17.10.1-2.tar.gz
mkdir clickhouse-19.17.10.1-2
tar -zxvf clickhouse-19.17.10.1-2.tar.gz -C clickhouse-19.17.10.1-2
```

#### 下载服务端脚本包

```bash
wget https://repo.yandex.ru/clickhouse/rpm/stable/x86_64/clickhouse-client-19.17.10.1-2.noarch.rpm
```

#### 下载客户端脚本包

```bash
wget https://repo.yandex.ru/clickhouse/rpm/stable/x86_64/clickhouse-server-19.17.10.1-2.noarch.rpm
```

#### 安装&配置

```bash
# 前提配置
/bin/cp -rf /home/clickhouse/clickhouse-19.17.10.1-2/clickhouse /usr/bin/
/bin/cp -rf /home/clickhouse/clickhouse-19.17.10.1-2/clickhouse-server /etc/init.d/
/bin/cp -rf /home/clickhouse/clickhouse-19.17.10.1-2/server/clickhouse.limits /etc/security/limits.d/clickhouse.conf
ln -s /usr/bin/clickhouse /usr/bin/clickhouse-extract-from-config
# cp clickhouse-odbc-bridge /usr/bin # 缺少这个
ln -s /usr/bin/clickhouse /usr/bin/clickhouse-odbc-bridge # 试试
# 安装服务端客户端脚本
rpm -ivh clickhouse-server-19.17.10.1-2.noarch.rpm --nodeps --force
rpm -ivh clickhouse-client-19.17.10.1-2.noarch.rpm --nodeps --force
```

#### 个性化配置

```shell
# 修改数据/日志目录
mkdir + chown
# 配置hdfs高可用
# 集群模式配置
# 网络配置
```

#### 登录退出

```bash
# 启动服务
service clickhouse-server start|stop|status|restart
# 客户端登录
clickhouse client -m
```

#### 重要配置文件

```bash
# 客户端配置文件
vim /etc/clickhouse-client/config.xml
# 服务端配置文件
vim /etc/clickhouse-server/config.xml
# 用户管理
vim /etc/clickhouse-server/users.xml
# 启动脚本
vim /etc/init.d/clickhouse-server
# 文件句柄
vi /etc/security/limits.d/clickhouse.conf
```

#### 卸载

```shell
# 停止服务
service clickhouse-server status
service clickhouse-server stop
# 备份配置文件
mkdir /home/clickhouse/config_bak_20201119
cp /etc/clickhouse-server/*.xml /home/clickhouse/config_bak_20201119
# 卸载服务端客户端脚本
rpm -e clickhouse-server-19.17.10.1-2.noarch --nodeps
rpm -e clickhouse-client-19.17.10.1-2.noarch
rm -rf /usr/bin/clickhouse*
rm -rf /etc/security/limits.d/clickhouse.conf
rm -rf /var/lib/clickhouse
rm -rf /etc/clickhouse-server/preprocessed
rm -rf /etc/clickhouse-server/config.d
rm -rf /etc/clickhouse-server/users.d
# 下载新包
cd /home/clickhouse/clickhouse-20.3-lts
# 安装新版本
rpm -ivh /home/clickhouse/clickhouse-20.3-lts/clickhouse-common-static-dbg-20.3.21.2-2.x86_64.rpm
rpm -ivh /home/clickhouse/clickhouse-20.3-lts/clickhouse-common-static-20.3.21.2-2.x86_64.rpm
rpm -ivh /home/clickhouse/clickhouse-20.3-lts/clickhouse-server-20.3.21.2-2.noarch.rpm
rpm -ivh /home/clickhouse/clickhouse-20.3-lts/clickhouse-client-20.3.21.2-2.noarch.rpm
# 修改配置文件(数据目录/用户密码/监听host/hdfsha/prom监控)
# 启动服务
service clickhouse-server start
systemctl start clickhouse-server
```



