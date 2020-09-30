```bash
#!/bin/bash
# 密钥对不存在则创建密钥
[ ! -f /root/.ssh/id_rsa.pub ] && ssh-keygen -t rsa -p '' &>/dev/null  
while read line;do
		# 提取文件中的ip
        ip=`echo $line | cut -d " " -f1` 
        # 提取文件中的用户名
        user_name=`echo $line | cut -d " " -f2` 
        # 提取文件中的密码
        pass_word=`echo $line | cut -d " " -f3`    
        
# expect 实现自动输入密码
expect <<EOF
		# 复制公钥到目标主机
        spawn ssh-copy-id -i /root/.ssh/id_rsa.pub $user_name@$ip   
        expect {
                "yes/no" { send "yes\n";exp_continue}     
                "password" { send "$pass_word\n"}
        }
        expect eof
EOF
# 读取存储ip的文件
done < /root/host_ip.txt      
# 推送你在目标主机进行的部署配置
pscp.pssh -h /root/host_ip.txt /root/your_scripts.sh /root   
# 进行远程配置，执行你的配置脚本
pssh -h /root/host_ip.txt -i bash /root/your_scripts.sh        
```

