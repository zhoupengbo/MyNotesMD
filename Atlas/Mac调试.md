```shell
# 启动HBase
sudo sh ./bin/start-hbase.sh
# 访问webui
http://localhost:61510
# 启动Solr
sudo ./bin/solr start -c -z localhost:2181 -p 8983 -force
# 访问webui
http://localhost:8983/
# Mac调试
## Main Class
org.apache.atlas.Atlas
## VM options
-Datlas.home=/Applications/bangong/atlas/apache-atlas/deploy 
-Datlas.conf=/Applications/bangong/atlas/apache-atlas/deploy/conf 
-Datlas.data=/Applications/bangong/atlas/apache-atlas/deploy/data 
-Datlas.log.dir=/Applications/bangong/atlas/apache-atlas/deploy/log 
-Dembedded.solr.directory=/Applications/bangong/atlas/apache-atlas/deploy/data/solr
## Program arguments
--port 31000 --app /Applications/bangong/atlas/apache-atlas/deploy/webapp/target/atlas-webapp-2.1.0
## module 
atlas-webapp
# 访问地址
http://localhost:31000/
```

参考：https://blog.csdn.net/Milkcoffeezhu/article/details/107027423