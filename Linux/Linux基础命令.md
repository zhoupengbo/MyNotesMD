#### 1. 关闭SELinux

```shell
# 获取SELinux状态
getenforce|sestatus|sestatus -v
# 临时关闭
setenforce 0
# 永久关闭
sed --follow-symlinks -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

#### 2. 检查字符集

```shell
echo $LANG
```

#### 3. 创建用户赋权

```shell
groupadd zhoupb
useradd zhoupb -r -m -g zhoupb
passwd zhoupb
chown -R zhoupb:zhoupb /usr/local/zhoupb*
```

#### 4. 查看IO

```shell
pidstat -d 1  # 展示I/O统计，每秒更新一次
```

#### 5. 查看带宽

```shell
ifconfig; ethtool bond0 | grep Speed
```

#### 6. 查看CPU信息

````shell
# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
# 查看CPU信息（型号）
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
````

#### 7. 查看那个磁盘设备IO高

```shell
iostat -x 1
```

#### 8. 查看内核版本

```shell 
uname -srm
uname -a
```

#### 9. 为普通用户添加sudo权限

```shell
# 添加写权限
chmod u+w /etc/sudoers
# 赋权
echo "clickhouse    ALL=(ALL)    NOPASSWD: ALL" >> /etc/sudoers
# 删除写权限
chmod u-w /etc/sudoers
```

#### 10. RPM 操作

```shell
# 安装
rpm -ivh xxx.rpm
# 强制安装
rpm -ivh xxx.rpm --nodeps --force
# 卸载
rpm -e --nodeps xxx
# 查看包内容
rpm -qpl xxx.rpm
# 查看已安装rpm包
rpm -qa | grep xxx
```

#### 11. 防火墙

```shell
# 检查状态
systemctl status firewalld
# 开启
systemctl start firewalld
# 关闭
systemctl stop firewalld
```

#### 12. SSH 免密

```shell
# 生成秘钥
ssh-keygen
# 使用rsa算法进行加密
ssh-keygen -t rsa
# 最小化安装没有ssh-copy-id，需要安装
yum  install  -y openssl-clients
# 发送秘钥
ssh-copy-id -i ~/.ssh/id_rsa.pub user@server
# 指定端口
ssh-copy-id -i ~/.ssh/id_rsa.pub "-p 10022 user@server"
```

#### 13. 临时修改主机名

```shell
hostname test.zpb.com
```

