[TOC]



### 一、测试方法和结论

​			Star schema benchmark（以下简称SSB）是学术界和工业界广泛使用的一个星型模型测试集（来源[论文](https://www.cs.umb.edu/~xuedchen/research/publications/StarSchemaB.PDF)），通过这个测试集合可以方便的对比各种OLAP产品的基础性能指标。Clickhouse 通过改写SSB，将星型模型打平转化成宽表，改造成了一个单表测试benchmark（参考[链接](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)）。本报告记录了DorisDB、ApacheDoris和Clickhouse在SSB单表和多表数据集上进行了性能对比测试的结果。测试结论如下：



### 二、测试准备

### 2.1 硬件环境

| 机器     | 1台物理机                                                    |
| -------- | ------------------------------------------------------------ |
| CPU      | **40** cores Intel(R)  Xeon(R) Silver 4210 CPU @ **2.20GHz** cache size: level1=32k,level2=1024k,level3=14080K |
| 内存     | 251GB                                                        |
| 网络带宽 | 1000Mb/s                                                     |
| 磁盘     | 5T                                                           |

### 2.2 软件环境

DorisDB部署在单节点物理机，。

内核版本：Linux 3.10.0-1127.el7.x86_64

操作系统版本：CentOS Linux release 7.8.2003 (Core)

软件版本：DorisDB-EE-1.11.10 企业版、ClickHouse server version 20.3.21.2 (official build).

### 三、测试数据与结果

### 3.1 测试数据

| 表名           | 行数     | 解释            |
| -------------- | -------- | --------------- |
| lineorder      | 6亿3万   | SSB商品订单表   |
| customer       | 300万    | SSB客户表       |
| part           | 140万    | SSB 零部件表    |
| supplier       | 20万     | SSB 供应商表    |
| dates          | 2556     | 日期表          |
| lineorder_flat | 9亿6千万 | SSB打平后的宽表 |

### 3.2 测试SQL

DorisDB单表测试SQL（执行3次取平均耗时）

```sql
--Q1.1 耗时：0.45s
SELECT sum(lo_extendedprice * lo_discount) AS `revenue` 
FROM lineorder_flat 
WHERE lo_orderdate >= '1993-01-01' and lo_orderdate <= '1993-12-31' AND lo_discount BETWEEN 1 AND 3 AND lo_quantity < 25; 
 
--Q1.2 耗时：0.36s
SELECT sum(lo_extendedprice * lo_discount) AS revenue FROM lineorder_flat  
WHERE lo_orderdate >= '1994-01-01' and lo_orderdate <= '1994-01-31' AND lo_discount BETWEEN 4 AND 6 AND lo_quantity BETWEEN 26 AND 35; 
 
--Q1.3 耗时：0.43s
SELECT sum(lo_extendedprice * lo_discount) AS revenue 
FROM lineorder_flat 
WHERE weekofyear(lo_orderdate) = 6 AND lo_orderdate >= '1994-01-01' and lo_orderdate <= '1994-12-31' 
 AND lo_discount BETWEEN 5 AND 7 AND lo_quantity BETWEEN 26 AND 35; 
 
 
--Q2.1 耗时：0.87s
SELECT sum(lo_revenue), year(lo_orderdate) AS year,  p_brand 
FROM lineorder_flat 
WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA' 
GROUP BY year,  p_brand 
ORDER BY year, p_brand; 
 
--Q2.2 耗时：1.33s
SELECT 
sum(lo_revenue), year(lo_orderdate) AS year, p_brand 
FROM lineorder_flat 
WHERE p_brand >= 'MFGR#2221' AND p_brand <= 'MFGR#2228' AND s_region = 'ASIA' 
GROUP BY year,  p_brand 
ORDER BY year, p_brand; 
  
--Q2.3 耗时：0.6s
SELECT sum(lo_revenue),  year(lo_orderdate) AS year, p_brand 
FROM lineorder_flat 
WHERE p_brand = 'MFGR#2239' AND s_region = 'EUROPE' 
GROUP BY  year,  p_brand 
ORDER BY year, p_brand; 
 
 
--Q3.1 耗时：0.6s
SELECT c_nation, s_nation,  year(lo_orderdate) AS year, sum(lo_revenue) AS revenue FROM lineorder_flat 
WHERE c_region = 'ASIA' AND s_region = 'ASIA' AND lo_orderdate  >= '1992-01-01' AND lo_orderdate   <= '1997-12-31' 
GROUP BY c_nation, s_nation, year 
ORDER BY  year ASC, revenue DESC; 
 
--Q3.2 耗时：1.92s
SELECT  c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue
FROM lineorder_flat 
WHERE c_nation = 'UNITED STATES' AND s_nation = 'UNITED STATES' AND lo_orderdate  >= '1992-01-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 
 
--Q3.3 耗时：0.59s
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue 
FROM lineorder_flat 
WHERE c_city in ( 'UNITED KI1' ,'UNITED KI5') AND s_city in ( 'UNITED KI1' ,'UNITED KI5') AND lo_orderdate  >= '1992-01-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 
 
--Q3.4 耗时：0.32s
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue 
FROM lineorder_flat 
WHERE c_city in ('UNITED KI1', 'UNITED KI5') AND s_city in ( 'UNITED KI1',  'UNITED KI5') AND  lo_orderdate  >= '1997-12-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city,  s_city, year 
ORDER BY year ASC, revenue DESC; 
 
 
--Q4.1 耗时：1.25s
-- set vectorized_engine_enable = FALSE; 
SELECT year(lo_orderdate) AS year, c_nation,  sum(lo_revenue - lo_supplycost) AS profit FROM lineorder_flat 
WHERE c_region = 'AMERICA' AND s_region = 'AMERICA' AND p_mfgr in ( 'MFGR#1' , 'MFGR#2') 
GROUP BY year, c_nation 
ORDER BY year ASC, c_nation ASC; 
 
--Q4.2 耗时：0.81s
SELECT year(lo_orderdate) AS year, 
    s_nation, p_category, sum(lo_revenue - lo_supplycost) AS profit 
FROM lineorder_flat 
WHERE c_region = 'AMERICA' AND s_region = 'AMERICA' AND lo_orderdate >= '1997-01-01' and lo_orderdate <= '1998-12-31' AND  p_mfgr in ( 'MFGR#1' , 'MFGR#2') 
GROUP BY year, s_nation,  p_category 
ORDER BY  year ASC, s_nation ASC, p_category ASC; 
 
--Q4.3 耗时：0.65s
SELECT year(lo_orderdate) AS year, s_city, p_brand, 
    sum(lo_revenue - lo_supplycost) AS profit 
FROM lineorder_flat 
WHERE s_nation = 'UNITED STATES' AND lo_orderdate >= '1997-01-01' and lo_orderdate <= '1998-12-31' AND p_category = 'MFGR#14' 
GROUP BY  year,  s_city, p_brand 
ORDER BY year ASC,  s_city ASC,  p_brand ASC; 
```

多表测试SQL

```sql
--Q1.1 耗时：0.55s
select sum(lo_revenue) as revenue
from lineorder join dates on lo_orderdate = d_datekey
where d_year = 1993 and lo_discount between 1 and 3 and lo_quantity < 25;

--Q1.2 耗时：0.47s
select sum(lo_revenue) as revenue
from lineorder
join dates on lo_orderdate = d_datekey
where d_yearmonthnum = 199401
and lo_discount between 4 and 6
and lo_quantity between 26 and 35;

--Q1.3 耗时：0.46s
select sum(lo_revenue) as revenue
from lineorder
join dates on lo_orderdate = d_datekey
where d_weeknuminyear = 6 and d_year = 1994
and lo_discount between 5 and 7
and lo_quantity between 26 and 35;


--Q2.1 耗时：6.24s
select sum(lo_revenue) as lo_revenue, d_year, p_brand
from lineorder
inner join dates on lo_orderdate = d_datekey
join part on lo_partkey = p_partkey
join supplier on lo_suppkey = s_suppkey
where p_category = 'MFGR#12' and s_region = 'AMERICA'
group by d_year, p_brand
order by d_year, p_brand;

--Q2.2 耗时：4.07s
select sum(lo_revenue) as lo_revenue, d_year, p_brand
from lineorder
join dates on lo_orderdate = d_datekey
join part on lo_partkey = p_partkey
join supplier on lo_suppkey = s_suppkey
where p_brand between 'MFGR#2221' and 'MFGR#2228' and s_region = 'ASIA'
group by d_year, p_brand
order by d_year, p_brand;

--Q2.3 耗时：3.24
select sum(lo_revenue) as lo_revenue, d_year, p_brand
from lineorder
join dates on lo_orderdate = d_datekey
join part on lo_partkey = p_partkey
join supplier on lo_suppkey = s_suppkey
where p_brand = 'MFGR#2239' and s_region = 'EUROPE'
group by d_year, p_brand
order by d_year, p_brand;


--Q3.1 耗时：12.78s
select c_nation, s_nation, d_year, sum(lo_revenue) as lo_revenue
from lineorder
join dates on lo_orderdate = d_datekey
join customer on lo_custkey = c_custkey
join supplier on lo_suppkey = s_suppkey
where c_region = 'ASIA' and s_region = 'ASIA'and d_year >= 1992 and d_year <= 1997
group by c_nation, s_nation, d_year
order by d_year asc, lo_revenue desc;

--Q3.2 耗时：5.37
select c_city, s_city, d_year, sum(lo_revenue) as lo_revenue
from lineorder
join dates on lo_orderdate = d_datekey
join customer on lo_custkey = c_custkey
join supplier on lo_suppkey = s_suppkey
where c_nation = 'UNITED STATES' and s_nation = 'UNITED STATES'
and d_year >= 1992 and d_year <= 1997
group by c_city, s_city, d_year
order by d_year asc, lo_revenue desc;

--Q3.3 耗时：3.91s
select c_city, s_city, d_year, sum(lo_revenue) as lo_revenue
from lineorder
join dates on lo_orderdate = d_datekey
join customer on lo_custkey = c_custkey
join supplier on lo_suppkey = s_suppkey
where (c_city='UNITED KI1' or c_city='UNITED KI5')
and (s_city='UNITED KI1' or s_city='UNITED KI5')
and d_year >= 1992 and d_year <= 1997
group by c_city, s_city, d_year
order by d_year asc, lo_revenue desc;

--Q3.4 耗时：0.62s
select c_city, s_city, d_year, sum(lo_revenue) as lo_revenue
from lineorder
join dates on lo_orderdate = d_datekey
join customer on lo_custkey = c_custkey
join supplier on lo_suppkey = s_suppkey
where (c_city='UNITED KI1' or c_city='UNITED KI5') and (s_city='UNITED KI1' or s_city='UNITED KI5') and d_yearmonth
 = 'Dec1997'
group by c_city, s_city, d_year
order by d_year asc, lo_revenue desc;


--Q4.1 耗时：12.87s
select d_year, c_nation, sum(lo_revenue) - sum(lo_supplycost) as profit
from lineorder
join dates on lo_orderdate = d_datekey
join customer on lo_custkey = c_custkey
join supplier on lo_suppkey = s_suppkey
join part on lo_partkey = p_partkey
where c_region = 'AMERICA' and s_region = 'AMERICA' and (p_mfgr = 'MFGR#1' or p_mfgr = 'MFGR#2')
group by d_year, c_nation
order by d_year, c_nation;

--Q4.2 耗时：3.56
select d_year, s_nation, p_category, sum(lo_revenue) - sum(lo_supplycost) as profit
from lineorder
join dates on lo_orderdate = d_datekey
join customer on lo_custkey = c_custkey
join supplier on lo_suppkey = s_suppkey
join part on lo_partkey = p_partkey
where c_region = 'AMERICA'and s_region = 'AMERICA'
and (d_year = 1997 or d_year = 1998)
and (p_mfgr = 'MFGR#1' or p_mfgr = 'MFGR#2')
group by d_year, s_nation, p_category
order by d_year, s_nation, p_category;

--Q4.3 耗时：1.89s
select d_year, s_city, p_brand, sum(lo_revenue) - sum(lo_supplycost) as profit
from lineorder
join dates on lo_orderdate = d_datekey
join customer on lo_custkey = c_custkey
join supplier on lo_suppkey = s_suppkey
join part on lo_partkey = p_partkey
where c_region = 'AMERICA'and s_nation = 'UNITED STATES'
and (d_year = 1997 or d_year = 1998)
and p_category = 'MFGR#14'
group by d_year, s_city, p_brand
order by d_year, s_city, p_brand;
```

ClickHouse单表测试SQL（执行3次取平均耗时）

```sql
-- Q1.1
SELECT sum(LO_EXTENDEDPRICE * LO_DISCOUNT) AS revenue FROM lineorder_flat WHERE toYear(LO_ORDERDATE) = 1993 AND LO_DISCOUNT BETWEEN 1 AND 3 AND LO_QUANTITY < 25;

-- Q1.2
SELECT sum(LO_EXTENDEDPRICE * LO_DISCOUNT) AS revenue FROM lineorder_flat WHERE toYYYYMM(LO_ORDERDATE) = 199401 AND LO_DISCOUNT BETWEEN 4 AND 6 AND LO_QUANTITY BETWEEN 26 AND 35;

-- Q1.3
SELECT sum(LO_EXTENDEDPRICE * LO_DISCOUNT) AS revenue FROM lineorder_flat WHERE toISOWeek(LO_ORDERDATE) = 6 AND toYear(LO_ORDERDATE) = 1994 AND LO_DISCOUNT BETWEEN 5 AND 7 AND LO_QUANTITY BETWEEN 26 AND 35;

-- Q2.1
SELECT sum(LO_REVENUE), toYear(LO_ORDERDATE) AS year, P_BRAND FROM lineorder_flat WHERE P_CATEGORY = 'MFGR#12' AND S_REGION = 'AMERICA' GROUP BY year, P_BRAND ORDER BY year, P_BRAND;

-- Q2.2
SELECT sum(LO_REVENUE), toYear(LO_ORDERDATE) AS year, P_BRAND FROM lineorder_flat WHERE P_BRAND >= 'MFGR#2221' AND P_BRAND <= 'MFGR#2228' AND S_REGION = 'ASIA' GROUP BY year, P_BRAND ORDER BY year, P_BRAND;

-- Q2.3
SELECT sum(LO_REVENUE), toYear(LO_ORDERDATE) AS year, P_BRAND FROM lineorder_flat WHERE P_BRAND = 'MFGR#2239' AND S_REGION = 'EUROPE' GROUP BY year, P_BRAND ORDER BY year, P_BRAND;

-- Q3.1
SELECT C_NATION, S_NATION, toYear(LO_ORDERDATE) AS year, sum(LO_REVENUE) AS revenue FROM lineorder_flat WHERE C_REGION = 'ASIA' AND S_REGION = 'ASIA' AND year >= 1992 AND year <= 1997 GROUP BY C_NATION, S_NATION, year ORDER BY year ASC, revenue DESC;

-- Q3.2
SELECT C_CITY, S_CITY, toYear(LO_ORDERDATE) AS year, sum(LO_REVENUE) AS revenue FROM lineorder_flat WHERE C_NATION = 'UNITED STATES' AND S_NATION = 'UNITED STATES' AND year >= 1992 AND year <= 1997 GROUP BY C_CITY, S_CITY, year ORDER BY year ASC, revenue DESC;

-- Q3.3
SELECT C_CITY, S_CITY, toYear(LO_ORDERDATE) AS year, sum(LO_REVENUE) AS revenue FROM lineorder_flat WHERE (C_CITY = 'UNITED KI1' OR C_CITY = 'UNITED KI5') AND (S_CITY = 'UNITED KI1' OR S_CITY = 'UNITED KI5') AND year >= 1992 AND year <= 1997 GROUP BY C_CITY, S_CITY, year ORDER BY year ASC, revenue DESC;

-- Q3.4
SELECT C_CITY, S_CITY, toYear(LO_ORDERDATE) AS year, sum(LO_REVENUE) AS revenue FROM lineorder_flat WHERE (C_CITY = 'UNITED KI1' OR C_CITY = 'UNITED KI5') AND (S_CITY = 'UNITED KI1' OR S_CITY = 'UNITED KI5') AND toYYYYMM(LO_ORDERDATE) = 199712 GROUP BY C_CITY, S_CITY, year ORDER BY year ASC, revenue DESC;

-- Q4.1
SELECT toYear(LO_ORDERDATE) AS year, C_NATION, sum(LO_REVENUE - LO_SUPPLYCOST) AS profit FROM lineorder_flat WHERE C_REGION = 'AMERICA' AND S_REGION = 'AMERICA' AND (P_MFGR = 'MFGR#1' OR P_MFGR = 'MFGR#2') GROUP BY year, C_NATION ORDER BY year ASC, C_NATION ASC;

-- Q4.2
SELECT toYear(LO_ORDERDATE) AS year, S_NATION, P_CATEGORY, sum(LO_REVENUE - LO_SUPPLYCOST) AS profit FROM lineorder_flat WHERE C_REGION = 'AMERICA' AND S_REGION = 'AMERICA' AND (year = 1997 OR year = 1998) AND (P_MFGR = 'MFGR#1' OR P_MFGR = 'MFGR#2') GROUP BY year, S_NATION, P_CATEGORY ORDER BY year ASC, S_NATION ASC, P_CATEGORY ASC;

-- Q4.3
SELECT toYear(LO_ORDERDATE) AS year, S_CITY, P_BRAND, sum(LO_REVENUE - LO_SUPPLYCOST) AS profit FROM lineorder_flat WHERE S_NATION = 'UNITED STATES' AND (year = 1997 OR year = 1998) AND P_CATEGORY = 'MFGR#14' GROUP BY year, S_CITY, P_BRAND ORDER BY year ASC, S_CITY ASC, P_BRAND ASC;
```

### 四、测试结果

### 4.1单表冷启动查询耗时(ms)，3次采样取平均耗时

| Sql编号 | DorisDB耗时(ms) | ClickHouse耗时(ms) | ClickHouse/DorisDB耗时比例 |
| ------- | --------------- | ------------------ | -------------------------- |
| Q1.1    | 164             | 214                | 1.3                        |
| Q1.2    | 122             | 44                 | 0.36                       |
| Q1.3    | 178             | 56                 | 0.31                       |
| Q2.1    | 669             | 1058               | 1.58                       |
| Q2.2    | 860             | 814                | 0.95                       |
| Q2.3    | 411             | 733                | 1.78                       |
| Q3.1    | 1171            | 2096               | 1.79                       |
| Q3.2    | 510             | 1689               | 3.31                       |
| Q3.3    | 398             | 1572               | 3.95                       |
| Q3.4    | 114             | 51                 | 0.45                       |
| Q4.1    | 859             | 1248               | 1.45                       |
| Q4.2    | 380             | 679                | 1.79                       |
| Q4.3    | 290             | 593                | 2.04                       |

![image-20210217174500067](/Users/weilinzi/Library/Application Support/typora-user-images/image-20210217174500067.png)

### 4.2单表热启动查询耗时(ms)，3次采样取平均耗时

| Sql编号 | DorisDB耗时(ms) | ClickHouse耗时(ms) | ClickHouse/DorisDB耗时比例 |
| ------- | --------------- | ------------------ | -------------------------- |
| Q1.1    | 166             | 216                | 1.3                        |
| Q1.2    | 118             | 44                 | 0.37                       |
| Q1.3    | 143             | 35                 | 0.24                       |
| Q2.1    | 584             | 969                | 1.66                       |
| Q2.2    | 845             | 774                | 0.92                       |
| Q2.3    | 411             | 744                | 1.81                       |
| Q3.1    | 1229            | 2088               | 1.7                        |
| Q3.2    | 505             | 1700               | 3.37                       |
| Q3.3    | 378             | 1526               | 4.04                       |
| Q3.4    | 114             | 48                 | 0.42                       |
| Q4.1    | 858             | 1306               | 1.52                       |
| Q4.2    | 384             | 675                | 1.76                       |
| Q4.3    | 288             | 587                | 2.04                       |

![image-20210217174548176](/Users/weilinzi/Library/Application Support/typora-user-images/image-20210217174548176.png)

### 4.3 单表压测

线程数：100

持续时间：3分钟

超时时间：2s

| Sql编号 | 引擎       | QPS  | 样本数量 | 平均值 | 中位数 | 90% 百分位 | 95% 百分位 | 99% 百分位 | 最小值 | 最大值 |
| ------- | ---------- | ---- | -------- | ------ | ------ | ---------- | ---------- | ---------- | ------ | ------ |
| Q1.1    | ClickHouse | 92   | 100      | 548    | 556    | 937        | 992        | 1010       | 8      | 1032   |
| Q1.1    | DorisDB    | 9    | 100      | 5300   | 3145   | 11374      | 11413      | 11466      | 2297   | 11480  |
| Q1.2    | ClickHouse | 109  | 299      | 736    | 552    | 1771       | 1888       | 2005       | 53     | 2006   |
| Q1.2    | DorisDB    | 7    | 100      | 5832   | 3253   | 13126      | 13292      | 13384      | 2445   | 13632  |
| Q1.3    | ClickHouse | 217  | 13717    | 443    | 481    | 563        | 590        | 867        | 3      | 1346   |
| Q1.3    | DorisDB    | 8    | 100      | 5454   | 3099   | 10243      | 12597      | 12640      | 2413   | 12644  |
| Q2.1    | ClickHouse | 48   | 100      | 1554   | 1542   | 1962       | 2003       | 2011       | 1038   | 2011   |
| Q2.1    | DorisDB    | 8    | 100      | 4701   | 3162   | 6446       | 7733       | 10543      | 2223   | 12816  |
| Q2.2    | ClickHouse | 45   | 100      | 1622   | 1637   | 1983       | 2013       | 2018       | 1095   | 2022   |
| Q2.2    | DorisDB    | 10   | 100      | 6384   | 6348   | 6648       | 6956       | 7375       | 5731   | 9248   |
| Q2.3    | ClickHouse | 83   | 100      | 655    | 660    | 1036       | 1110       | 1140       | 134    | 1140   |
| Q2.3    | DorisDB    | 13   | 100      | 6393   | 6409   | 6667       | 6698       | 7008       | 5782   | 7351   |
| Q3.1    | ClickHouse | 47   | 100      | 1552   | 1587   | 1914       | 1989       | 2010       | 1040   | 2022   |
| Q3.1    | DorisDB    | 9    | 100      | 6030   | 6021   | 6369       | 6467       | 6946       | 5253   | 10003  |
| Q3.2    | ClickHouse | 46   | 100      | 1582   | 1566   | 1967       | 2007       | 2014       | 1110   | 2026   |
| Q3.2    | DorisDB    | 11   | 100      | 7668   | 7507   | 8566       | 8706       | 8931       | 6890   | 9112   |
| Q3.3    | ClickHouse | 47   | 100      | 1560   | 1567   | 1953       | 1990       | 2007       | 1052   | 2007   |
| Q3.3    | DorisDB    | 7    | 100      | 6234   | 4341   | 8871       | 9255       | 9874       | 3310   | 13012  |
| Q3.4    | ClickHouse | 94   | 348      | 880    | 976    | 1409       | 1656       | 1828       | 9      | 1927   |
| Q3.4    | DorisDB    | 10   | 100      | 6167   | 6118   | 6440       | 6536       | 7259       | 5360   | 9316   |
| Q4.1    | ClickHouse | 48   | 100      | 1562   | 1560   | 1952       | 2009       | 2012       | 1049   | 2013   |
| Q4.1    | DorisDB    | 12   | 100      | 6218   | 6170   | 6451       | 6528       | 7884       | 5488   | 8381   |
| Q4.2    | ClickHouse | 47   | 100      | 1558   | 1552   | 1944       | 2002       | 2011       | 1065   | 2012   |
| Q4.2    | DorisDB    | 7    | 100      | 6684   | 5190   | 8541       | 9722       | 9771       | 4212   | 14344  |
| Q4.3    | ClickHouse | 48   | 100      | 1561   | 1546   | 1957       | 2007       | 2009       | 1055   | 2014   |
| Q4.3    | DorisDB    | 14   | 100      | 5966   | 6002   | 6248       | 6328       | 6448       | 5110   | 6951   |

![image-20210225172422576](/Users/weilinzi/Library/Application Support/typora-user-images/image-20210225172422576.png)

### 单表测试总结

1. **单表测试的13个查询中（冷热启动），4个查询Clickhouse速度最快，另外，有9个查询DorisDB查询速度最快**
2. **单表测试中，13个查询Clickhouse总耗时(冷热启动)是DorisDB 的1.77倍**
3. **单表测试，ClickHouse和DorisDB都能满足Serving要求**



### 单表压测总结

1. **单表压测的13个查询中，所有查询Clickhouse速度最快，CH的最高QPS是DorisDB的15倍**
2. **单表压测中，ClickHouse查询耗时都小于2s，能满足Serving要求；DorisDB查询都大于3s，可以满足OLAP要求**



### 4.4 CH、DorisDB多表测试

DorisDB多表测试SQL(线上业务表)

```sql
-- 多表测试SQL(线上业务表)
-- Q1.1
select a.city_name, sum(d.hire_cnt) from ssb.dwd_cust_all_customer_f_d a left join ssb.ads_hire_cnt_f_d d on a.keeper_code = d.keeper_code group by a.city_name order by city_name;
-- Q1.2
select count(1) from ssb.dwd_cust_all_customer_f_d a left join ssb.ads_hire_cnt_f_d d on a.keeper_code = d.keeper_code;
-- Q1.3
select a.city_name, sum(d.hire_cnt), count(1) from ssb.dwd_cust_all_customer_f_d a left join ssb.ads_hire_cnt_f_d d on a.keeper_code = d.keeper_code where d.deptid_l3 in ('102815', '105406', '102814', '100944') group by a.city_name order by city_name;
```

ClickHouse多表测试SQL(线上业务表)

```sql
-- Q1.1
select a.city_name, sum(d.hire_cnt) from test.dwd_cust_all_customer_f_d a left join test.ads_hire_cnt_f_d d on a.keeper_code = d.keeper_code group by a.city_name order by city_name;
-- Q1.2
select count(1) from test.dwd_cust_all_customer_f_d a left join test.ads_hire_cnt_f_d d on a.keeper_code = d.keeper_code;
-- Q1.3
select a.city_name, sum(d.hire_cnt), count(1) from test.dwd_cust_all_customer_f_d a left join test.ads_hire_cnt_f_d d on a.keeper_code = d.keeper_code where d.deptid_l3 in ('102815', '105406', '102814', '100944') group by a.city_name order by city_name;
```

多表测试结果

线程数：10

持续时间：3分钟

超时时间：未设置

left join总数据量：6亿71万8155

| Sql编号 | DorisDB耗时(ms) | ClickHouse耗时(ms) | ClickHouse/DorisDB耗时对比 |
| ------- | --------------- | ------------------ | -------------------------- |
| Q1.1    | 12870           | 2849               | 0.11                       |
| Q1.2    | 12673           | 2248               | 0.18                       |
| Q1.3    | 1913            | 4118               | 2.15                       |

![image-20210226104928686](/Users/weilinzi/Library/Application Support/typora-user-images/image-20210226104928686.png)

多表压测结果

| Sql编号 | 引擎       | 中位数 | 吞吐量 | 吞吐量  | 样本数量 | 平均值 | 90% 百分位 | 95% 百分位 | 99% 百分位 | 最小值 | 最大值 |
| ------- | ---------- | ------ | ------ | ------- | -------- | ------ | ---------- | ---------- | ---------- | ------ | ------ |
| Q1.1    | ClickHouse | 19350  | 0.52   | 0.52124 | 160      | 19027  | 20888      | 21294      | 22397      | 8393   | 22479  |
| Q1.1    | DorisDB    | 16267  | 0.61   | 0.60607 | 190      | 16396  | 17317      | 17671      | 18736      | 14460  | 18831  |
| Q1.2    | ClickHouse | 6798   | 1.48   | 1.47932 | 451      | 6666   | 7187       | 7292       | 7475       | 3877   | 7640   |
| Q1.2    | DorisDB    | 8294   | 1.19   | 1.18947 | 365      | 8329   | 8831       | 9034       | 9641       | 7278   | 10397  |
| Q1.3    | ClickHouse | 28004  | 0.36   | 0.35634 | 110      | 27954  | 29437      | 30144      | 30878      | 21541  | 31677  |
| Q1.3    | DorisDB    | 2292   | 4.33   | 4.32629 | 1306     | 2298   | 2406       | 2447       | 2532       | 2016   | 3580   |

![image-20210226102805726](/Users/weilinzi/Library/Application Support/typora-user-images/image-20210226102805726.png)

### 多表测试总结

1. **多表测试的3个查询中，2个查询Clickhouse速度最快，1个查询DorisDB查询速度最快**
2. **多表测试中，13个查询DorisDB总耗时是Clickhouse的4.34倍**
3. **多表查询，ClickHouse查询耗时比较稳定，在2~5s之间，DorisDB不是很稳定，在1到13s之间**
4. **多表Join查询，总数据量过亿，Clickhouse和DorisDB的查询耗时最少也超过了2s，最大耗时超过12s，无法满足serving要求，基本可以满足OLAP要求**

### 多表压测总结

1. **多表压测的3个查询中，1个Clickhouse查询最快，2个DorisDB查询最快**
2. **多表压测的3个查询中，查询ClickHouse总耗时是DorisDB的2.02倍**
3. **多表Join压测，总数据量过亿，查询耗时最少也超过了2s，最大耗时超过28s，无法满足serving要求，基本可以满足OLAP要求**