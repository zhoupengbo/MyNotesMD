#### 常用语法

```sql
desc formatted table_name;
set hive.groupby.skewindata = true
```

#### 核心组件

核心：metastore/ql/serde

#### 分桶

Buckets：对列计算 hash，根据 hash 值切分数据，目的是为了并行。

#### 排序

- order by 是全局排序，只有一个reduce，数据量多时速度慢
- sort by 是随机分发到一个reduce然后reduce内部排序
- distribute by 是根据 distribute by 的字段把相应的记录分发到那个reduce
- cluster by是distribute by + sort by的简写

#### 优化

1. 扫描相关
   - 谓词下推(Predicate Push Down) 
   - 列剪裁(Column Pruning)
   - 分区剪裁(Partition Pruning)

2. 关联JOIN相关
   - Join操作左边为小表 
     - 位于 Join 操作 符左边的表的内容会被加载进内存
   - Join启动的job个数 
   - MapJoin

3. 分组Group By相关 

   - Skew In Data

     - hive.groupby.skewindata = true

       当选项设定为 true，生成的查询计划会有两个 MR Job。

       第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样 处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的;

       第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中(这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中)，最后完成最终的聚合操作。

4. 合并 小文件
5. 注意空值
   - 大量空值可能导致数据倾斜
   - 数据量再大，对Hive都不是问题，数据有一点斜，对Hive就是大问题