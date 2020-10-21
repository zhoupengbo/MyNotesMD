```bash
#!/bin/bash

# ps -ef|grep xxx |awk '{print $2}'|xargs kill -9
pid=$(ps -ef | grep xxx | grep -v grep | awk '{print $2}')
if [ $pid ];then 
    kill $pid
    echo "kill pid: $pid"
else
    echo "service is not running!"
fi
```

