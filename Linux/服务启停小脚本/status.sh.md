```bash
#! /bin/bash
pid=$(ps -ef | grep xxx | grep -v grep | awk '{print $2}')
if [ $pid ];then
    echo "status: running; pid: $pid" 
else 
    echo "status: stopped!"
fi
```

