一。新增数据

1. 每日数据写入新表（表名区别）
2. 写入完成后将表名替换为Merge引擎识别的表名

二。覆盖写

1. 每日数据写入新表（表名区别）
2. 将新表表名与旧表表名重命名
3. 新表生效，删除旧表



缺点：表数据量（Zookeeper元数据）激增。

