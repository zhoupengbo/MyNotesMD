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
yum install sysstat -y
iostat -x 1
```

#### 8. 查看系统和内核版本

```shell 
uname -srm
uname -a
cat /etc/redhat-release;uname -r
# 查看系统编码
echo $LANG
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
# linux暴露端口可以被外部访问: https://cloud.tencent.com/developer/article/1404152
/sbin/iptables -I INPUT -p tcp --dport 9092 -j ACCEPT


# 关闭iptables服务：
1、	systemctl stop iptables
2、	systemctl disable iptables
3、	systemctl status iptables

# 关闭Selinux
setenforce 0
vi /etc/sysconfig/selinux
# 输入小写i
# 确认SELINUX=disabled，如果不是则改成SELINUX=disabled
# 按:wq保存退出
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

#### 14. Top

```shell
# 按用户
top -u root
# 每隔2秒显示pid是12345的进程的资源使用情况，并显式该进程启动的命令行参数
top -d 2 -c -p 123456 
# 常用命令
P：按%CPU使用率排行
T：按累计时间排行
M：按%MEM排行
```

#### 15. 排查进程中各线程占用内存情况

```shell
# 定位线程问题,查看进程有哪些线程
ps p ${pid} -L -o pcpu,pmem,pid,tid,time,tname,cmd
# 查看总共有多少线程
ps p ${pid} -L -o pcpu,pmem,pid,tid,time,tname,cmd |wc -l
# 打印堆栈信息
jstack -l ${pid} > jstack.log
# 参考
https://www.cnblogs.com/z-sm/p/6745375.html
```

#### 16. Systemctl

```
systemctl daemon-reload    # 重载系统服务
systemctl enable *.service # 设置某服务开机启动      
systemctl start *.service  # 启动某服务  
systemctl stop *.service   # 停止某服务 
systemctl reload *.service # 重启某服务
```

#### 17. ps

```shell
# 显示所有包含其他使用者的行程及显示程序间的关系
ps -auxf 
# 显示指定用户
ps -u root 
# 显示所有进程信息，连同命令行
ps -ef 
```

