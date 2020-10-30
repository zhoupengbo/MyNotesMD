OPTIMIZE+ReplicatedReplacingMergeTree+View(版本控制)

对最新分区采用版本控制，通过Rename视图，手动暴露可对外显示的版本号。

导入数据完成后，执行OPTIMIZE触发合并，保证历史分区数据只有一个版本。

经测试：OPTIMIZE 操作不能保证所有节点进行数据去重合并，只针对单机。若要解决这个问题，需要保证相同的数据每次都落到同一个CH数据节点。

解决办法：

1. 制定数据路由规则，相同key的数据路由到相同的shard。
2. 使用distribute引擎写入，指定分片规则。