##### 修改配置文件 config.xml

```xml
<distributed_ddl>
  <!-- Path in ZooKeeper to queue with DDL queries -->
  <path>/clickhouse/task_queue/ddl</path>
  <cleanup_delay_period>60</cleanup_delay_period>
  <task_max_lifetime>86400</task_max_lifetime>
  <max_tasks_in_queue>200</max_tasks_in_queue>
</distributed_ddl>
```

##### 参数解析：

- cleanup_delay_period：检查DDL记录清理的间隔，单位为秒，默认60秒。

- task_max_lifetime：分布式DDL记录可以保留的最大时长，单位为秒，默认保留7天。

- max_tasks_in_queue：分布式DDL队列中可以保留的最大记录数，默认为1000条。

##### 源码位置：dbms/src/interpreters/DDLWorker.h



作者：LittleMagic
链接：https://www.jianshu.com/p/52a2d7e3a896
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

