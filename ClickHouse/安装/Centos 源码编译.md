#### 1. 检查是否支持SSE4.2

```shell
$ grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"
```

#### 2. 安装依赖项

```shell
$ yum install git cmake libicu-devel clang libicu-devel readline-devel mysql-devel openssl-devel unixODBC_devel bzip2 -y
```

#### 3. 安装高版本 gcc9

```shell
$ wget ftp://gnu.mirror.iweb.com/gcc/gcc-9.3.0/gcc-9.3.0.tar.xz
$ tar xvf gcc-9.3.0.tar.xz
$ cd gcc-9.3.0
$ ./contrib/download_prerequisites
```

在执行这一步时，`./contrib/download_prerequisites`，可能会遇到DNS无法解析，网络不通的问题。需要手动安装依赖包。

###### 3.1 手动下载以下依赖包

- [ftp://gcc.gnu.org/pub/gcc/infrastructure/isl-0.18.tar.bz2](ftp://gcc.gnu.org/pub/gcc/infrastructure/isl-0.18.tar.bz2)
- [ftp://gcc.gnu.org/pub/gcc/infrastructure/mpc-1.0.3.tar.gz](ftp://gcc.gnu.org/pub/gcc/infrastructure/mpc-1.0.3.tar.gz)
- [ftp://gcc.gnu.org/pub/gcc/infrastructure/mpfr-3.1.4.tar.bz2](ftp://gcc.gnu.org/pub/gcc/infrastructure/mpfr-3.1.4.tar.bz2)
- [ftp://gcc.gnu.org/pub/gcc/infrastructure/gmp-6.1.0.tar.bz2](ftp://gcc.gnu.org/pub/gcc/infrastructure/gmp-6.1.0.tar.bz2)

###### 3.2 解压

```shell
$ tar -xjf mpfr-3.1.4.tar.bz2 || exit 1
$ tar -xjf gmp-6.1.0.tar.bz2 || exit 1
$ tar -xzf mpc-1.0.3.tar.gz || exit 1
$ tar -xjf isl-0.18.tar.bz2 || exit 1
```

###### 3.3 将加压后的压缩包移动至gcc-9.3.0下并重命名

```shell
$ mv mpfr-3.1.4.tar.bz2 gcc-9.3.0/mpfr
$ mv gmp-6.1.0.tar.bz2 gcc-9.3.0/gmp
$ mv mpc-1.0.3.tar.gz gcc-9.3.0/mpc
$ mv isl-0.18.tar.bz2 gcc-9.3.0/isl
```

###### 3.4 继续构建

```shell
$ cd gcc-9.3.0 & mkdir build & cd build
$ ../configure --prefix=/opt/gcc9 --enable-languages=c,c++   --disable-multilib
$ export THREADS=$(grep -c ^processor /proc/cpuinfo)
$ make -j $THREADS
$ make install
```

###### 3.5 添加软连接

```shell
$ cd /opt/gcc9/bin/
$ ln -s gcc cc
$ ln -s g++ g++-9
$ ln -s gcc gcc-9
$ ln -s /opt/gcc9/bin/* /usr/local/bin/
```

###### 3.6 添加环境变量

```shell
$ vim ~/.bashrc
$ export GCC9_HOME=/opt/gcc9
$ export PATH=$GCC9_HOME/bin:$PATH
$ export CC=gcc-9
$ export CXX=g++-9
$ source ~/.bashrc
```

###### 3.7 验证版本

```shell
$ gcc --version
```

#### 4. 安装cmake 3

###### 4.1 准备软件

```shell
$ wget https://cmake.org/files/v3.14/cmake-3.14.5-Linux-x86_64.tar.gz
$ tar zxvf cmake-3.14.5-Linux-x86_64.tar.gz -C /opt
$ ln -s cmake-3.14.5-Linux-x86_64 cmake
```

###### 4.2 添加环境变量

```shell
$ vim /etc/profile
$ export CMAKE_HOME=/opt/cmake
$ export PATH=$CMAKE_HOME/bin:$PATH
$ source /etc/profile
```

#### 5. 安装ninja

```shell
$ wget https://github.com/ninja-build/ninja/releases/download/v1.9.0/ninja-linux.zip
$ unzip ninja-linux.zip -d /usr/local/bin/
  
# 测试版本：
$ ninja --version
 
$ rm -f /lib64/libstdc++.so.6
$ ln -s /opt/gcc9/lib64/libstdc++.so.6 /lib64/libstdc++.so.6
```

#### 6. 源码编译

```shell
$ git clone -b v19.17.10.1-stable --depth 1 --recursive https://github.com.cnpmjs.org/ClickHouse/ClickHouse.git  # 需要很长时间
 
$ cd ClickHouse
 
$ mkdir build
$ cd build
$ cmake ..
$ ninja clickhouse
```

编译成功，显示如下：

```shell
[3950/3950] Linking CXX executable programs/clickhouse
```

#### 7. 启动登录

```shell
# 拷贝配置文件
$ mkdir -p /usr/local/clickhouse/etc
$ mkdir -p /usr/local/clickhouse/bin
$ cp ./ClickHouse/programs/server users.xml config.xml /usr/local/clickhouse/etc
$ cp ./ClickHouse/build/programs/clickhouse /usr/local/clickhouse/bin
 
# 启动clickhouse-server
$ clickhouse server --config-file=/usr/local/clickhouse/etc/config.xml
 
# 启动客户端
$ clickhouse client
$ clickhouse client --host=example.com
```

#### 8. 打包

```
cp ./ClickHouse/debian/clickhouse.limits ./ClickHouse/build/programs/server/
cd ./ClickHouse/build/programs
tar -zcvf clickhouse-19.17.10.1-2.tar.gz ./* -C ./
```



###### 参考：

- https://clickhouse.tech/docs/en/engines/table-engines/integrations/hdfs/
- https://clickhouse.tech/docs/en/interfaces/formats/#formats
- https://clickhouse.tech/docs/en/sql-reference/table-functions/hdfs/
- https://github.com/jaykelin/clickhouse-hdfs-loader
- https://github.com/ClickHouse/ClickHouse/pull/7940 (bug修复)
- https://github.com/ClickHouse/ClickHouse/commit/edf50f3ce327a63e36a1bd4b14f6653f0687f688（PR）
- https://github.com/ClickHouse/ClickHouse/blob/v20.1.16.120-stable/contrib/libhdfs3-cmake/CMakeLists.txt



