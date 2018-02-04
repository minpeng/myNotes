## Hbase+Phoenix 优化


### Hbase 性能优化
1.预分配regin
2.启用压缩以减少HDFS数据量,可提高读性能
3.Region Server进程配置大内存(>16G <100G)
4.每个Regin Server拥有的region数量<200
5.优化表结构设计

### Phoenix 优化
> https://phoenix.apache.org/faq.html#Are_there_any_tips_for_optimizing_Phoenix


 1.盐值
 2.预切分
 ```
 CREATE TABLE IF NOT EXISTS "PHOENIX_TABLE" (
  "key" VARCHAR NOT NULL PRIMARY KEY,
  "info"."DataSetTYPE" VARCHAR,
  "info"."DATE" VARCHAR) 
  SPLIT ON ('CS','EU','NA')
 ```
 3.多列蔟

 4.压缩
 - GZ 算法
 - SNAPPY 算法
 ```
 CREATE TABLE IF NOT EXISTS "PHOENIX_TABLE" (
  "key" VARCHAR NOT NULL PRIMARY KEY,
  "info"."DataSetTYPE" VARCHAR,
  "info"."DATE" VARCHAR) 
  COMPRESSION='GZ
 ```

 5.索引
 - Covered Indexes
 ```
 只需要通过索引就能返回所要查询的数据，所以索引的列必须包含所需查询的列(SELECT的列和WHRER的列)
 ```
 - Functional Indexes
 - Global Indexes
 > 读多写少

 - Local Indexes
 
 1.写多读少
 2.当查询的字段不完全是索引字段时本地索引也会被使用
 3.所有的本地索引都单独存储在同一张共享表中，由于无法预先确定region的位置，所以在读取数据时会检查每个region上的数据因而带来一定性能开销
 

 6.Hbase集群参数优化
 7.Phoenix参数优化