异常日志：

```
Caused by: java.lang.UnsupportedOperationException: Constructor threw an exception for org.apache.hadoop.hbase.ipc.NettyRpcServer
	at org.apache.hadoop.hbase.util.ReflectionUtils.instantiate(ReflectionUtils.java:66)
	at org.apache.hadoop.hbase.util.ReflectionUtils.instantiateWithCustomCtor(ReflectionUtils.java:45)
	at org.apache.hadoop.hbase.ipc.RpcServerFactory.createRpcServer(RpcServerFactory.java:66)
	at org.apache.hadoop.hbase.master.MasterRpcServices.createRpcServer(MasterRpcServices.java:340)
	at org.apache.hadoop.hbase.regionserver.RSRpcServices.<init>(RSRpcServices.java:1231)
	at org.apache.hadoop.hbase.regionserver.RSRpcServices.<init>(RSRpcServices.java:1184)
	at org.apache.hadoop.hbase.master.MasterRpcServices.<init>(MasterRpcServices.java:328)
	at org.apache.hadoop.hbase.master.HMaster.createRpcServices(HMaster.java:690)
	at org.apache.hadoop.hbase.regionserver.HRegionServer.<init>(HRegionServer.java:561)
	at org.apache.hadoop.hbase.master.HMaster.<init>(HMaster.java:471)
	at org.apache.hadoop.hbase.master.HMasterCommandLine$LocalHMaster.<init>(HMasterCommandLine.java:308)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.apache.hadoop.hbase.util.JVMClusterUtil.createMasterThread(JVMClusterUtil.java:131)
	... 7 more
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.apache.hadoop.hbase.util.ReflectionUtils.instantiate(ReflectionUtils.java:58)
	... 22 more
Caused by: java.net.BindException: Can't assign requested address
	at sun.nio.ch.Net.bind0(Native Method)
	at sun.nio.ch.Net.bind(Net.java:444)
	at sun.nio.ch.Net.bind(Net.java:436)
	at sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:225)
	at org.apache.hbase.thirdparty.io.netty.channel.socket.nio.NioServerSocketChannel.doBind(NioServerSocketChannel.java:128)
	at org.apache.hbase.thirdparty.io.netty.channel.AbstractChannel$AbstractUnsafe.bind(AbstractChannel.java:558)
	at org.apache.hbase.thirdparty.io.netty.channel.DefaultChannelPipeline$HeadContext.bind(DefaultChannelPipeline.java:1283)
	at org.apache.hbase.thirdparty.io.netty.channel.AbstractChannelHandlerContext.invokeBind(AbstractChannelHandlerContext.java:501)
	at org.apache.hbase.thirdparty.io.netty.channel.AbstractChannelHandlerContext.bind(AbstractChannelHandlerContext.java:486)
	at org.apache.hbase.thirdparty.io.netty.channel.DefaultChannelPipeline.bind(DefaultChannelPipeline.java:989)
	at org.apache.hbase.thirdparty.io.netty.channel.AbstractChannel.bind(AbstractChannel.java:254)
	at org.apache.hbase.thirdparty.io.netty.bootstrap.AbstractBootstrap$2.run(AbstractBootstrap.java:364)
	at org.apache.hbase.thirdparty.io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java:163)
	at org.apache.hbase.thirdparty.io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:403)
	at org.apache.hbase.thirdparty.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:463)
	at org.apache.hbase.thirdparty.io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:858)
	at org.apache.hbase.thirdparty.io.netty.util.concurrent.DefaultThreadFactory$DefaultRunnableDecorator.run(DefaultThreadFactory.java:138)
	at java.lang.Thread.run(Thread.java:748)
```

解决：

这个错误  regionserver.HRegionServer: Failed construction RegionServer 的翻译结果是：构建区域服务器失败。

通过观察发现可能是主机 hostsname 没设定或者 host 解析失败导致（实际是 google 后发现大多都是这个问题）。

通过设置 hostsname  和修改 hosts  就能解决该问题。