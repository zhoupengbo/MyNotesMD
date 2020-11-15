```shell
# 查找入口（main函数）
find . -name '*.java' | xargs grep --color 'main(' | awk '{print $1}' | uniq | grep -v test
```

客户端主入口：CliDriver

源码分析入口：Driver.compile()

#### SQL执行顺序

一个SQL大致分为以下7部分，按顺序执行 ：

- (5)SELECT
- (6)DISTINCT <-select list>
- (1)FROM <-table source>
- (2)WHERE <-condition> 
- (3)GROUP BY <-group by list> 
- (4)HAVING <-having condition> 
- (7)ORDER BY <-order by list>

#### Operators对应SQL

| Operators          | 功能描述                       | 对应SQL关键字             |
| ------------------ | ------------------------------ | ------------------------- |
| TableScanOperator  | 从表中读取数据                 | (1) FROM                  |
| ReduceSinkOperator | 把数据发送给Reduce端作聚合操作 | 有JOIN/GROUP BY的会有     |
| JoinOperator       | 作两个表的关联操作Join         | JOIN                      |
| SelectOperator     | 选择部分列并输出               | (5) SELECT                |
| FileSinkOperator   | 建立结果数据并发送给文件       | 每个SQL都有               |
| FilterOperator     | 过滤输入数据                   | (2) WHERE (4) HAVING      |
| GroupByOperator    | 对数据作Group By操作           | (3) GROUP BY (6) DISTINCT |
| MapJoinOperator    | 在内存中作大表和小表的关联操作 | JOIN                      |
| LimitOperator      | 作limit，返回一定条目的数据    | LIMIT                     |
| UnionOperator      | 作union                        | UNION                     |

- 每个步骤对应一个逻辑运算符(Operator) 
- 每个Operator输出一个虚表(VirtualTable)
- Explain 可查看执行计划
- 对抗 Join 倾斜最有力的工具就是MapJoin。



