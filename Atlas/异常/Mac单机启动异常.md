#### 异常一：HBase连接zookeeper异常

```
java.net.ConnectException: Connection refused
        at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
        at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
        at org.apache.zookeeper.ClientCnxnSocketNIO.doTransport(ClientCnxnSocketNIO.java:361)
        at org.apache.zookeeper.ClientCnxn$SendThread.run(ClientCnxn.java:1081)
2017-11-30 17:14:39,594 WARN  - [main:] ~ Possibly transient ZooKeeper, quorum=localhost:2181, exception=org.apache.zookeeper.KeeperException$ConnectionLossException: KeeperErrorCode = Connecti$
2017-11-30 17:14:40,593 WARN  - [main-SendThread(localhost:2181):] ~ Session 0x0 for server null, unexpected error, closing socket connection and attempting reconnect (ClientCnxn$SendThread:110$
java.net.ConnectException: Connection refused
        at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
        at org.apache.zookeeper.ClientCnxnSocketNIO.doTransport(ClientCnxnSocketNIO.java:361)
        at org.apache.zookeeper.ClientCnxn$SendThread.run(ClientCnxn.java:1081)
2017-11-30 17:14:56,185 WARN  - [main:] ~ Possibly transient ZooKeeper, quorum=localhost:2181, exception=org.apache.zookeeper.KeeperException$ConnectionLossException: KeeperErrorCode = Connecti$
2017-11-30 17:14:56,186 ERROR - [main:] ~ ZooKeeper exists failed after 4 attempts (RecoverableZooKeeper:277)
2017-11-30 17:14:56,186 WARN  - [main:] ~ hconnection-0x1dba4e060x0, quorum=localhost:2181, baseZNode=/hbase Unable to set watcher on znode (/hbase/hbaseid) (ZKUtil:544)
org.apache.zookeeper.KeeperException$ConnectionLossException: KeeperErrorCode = ConnectionLoss for /hbase/hbaseid
        at org.apache.zookeeper.KeeperException.create(KeeperException.java:99)
        at org.apache.zookeeper.KeeperException.create(KeeperException.java:51)
        at org.apache.zookeeper.ZooKeeper.exists(ZooKeeper.java:1045)
        at org.apache.hadoop.hbase.zookeeper.RecoverableZooKeeper.exists(RecoverableZooKeeper.java:221)
        at org.apache.hadoop.hbase.zookeeper.ZKUtil.checkExists(ZKUtil.java:541)
        at org.apache.hadoop.hbase.zookeeper.ZKClusterId.readClusterIdZNode(ZKClusterId.java:65)
```

解决办法：

在conf/atlas-env.sh中配置 JAVA_HOME。

#### 异常二：Solr 连接异常

```
2020-12-23 20:49:17,017 WARN  - [main:] ~ Failed to obtain graph instance on attempt 3 of 3 (AtlasGraphProvider:118)
java.lang.IllegalArgumentException: Could not instantiate implementation: org.janusgraph.diskstorage.solr.Solr6Index
	at org.janusgraph.util.system.ConfigurationUtil.instantiate(ConfigurationUtil.java:64)
	at org.janusgraph.diskstorage.Backend.getImplementationClass(Backend.java:440)
	at org.janusgraph.diskstorage.Backend.getIndexes(Backend.java:427)
	at org.janusgraph.diskstorage.Backend.<init>(Backend.java:150)
	at org.janusgraph.graphdb.configuration.GraphDatabase
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.janusgraph.util.system.ConfigurationUtil.instantiate(ConfigurationUtil.java:58)
	... 97 more
Caused by: org.apache.solr.common.SolrException: Cannot connect to cluster at localhost:2181: cluster not found/not ready
	at org.apache.solr.common.cloud.ZkStateReader.createClusterStateWatchersAndUpdate(ZkStateReader.java:385)
	at org.apache.solr.client.solrj.impl.ZkClientClusterStateProvider.connect(ZkClientClusterStateProvider.java:141)
	at org.apache.solr.client.solrj.impl.CloudSolrClient.connect(CloudSolrClient.java:383)
	at org.janusgraph.diskstorage.solr.Solr6Index.createSolrClient(Solr6Index.java:286)
	at org.janusgraph.diskstorage.solr.Solr6Index.<init>(Solr6Index.java:209)
	... 102 more
```

