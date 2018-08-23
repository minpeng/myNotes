## Kudu


### hive-kudu映射

1.txt日志文件导入到hive中
2.创建kudu表
3.hive表初级插入kudu表

```
insert into kudu_table_name select * from hive_table_name

```

### impala表中的数据导入Kudu表

1.建立Kudu外部映射表
```
CREATE EXTERNAL TABLE my_mapping_table
STORED AS KUDU
TBLPROPERTIES (
  'kudu.table_name' = 'my_kudu_table'
);
```

### Kudu主键的限制:
1.表创建后主键不可更改
2.主键内容不可以update操作修改。要修改一行的主键值。先要删除并新增一行新数据
3.主键的类型不支持DOUBLE,FLOAT,BOOL,(主键必须非空)
4.自动生成的主键不支持
5.每行对应的主键存储单元最大为16KB

### Kudu 列的限制
1.不支持MySQL的部分数据类型(DECIMAL,CHAR,VARCHAR,DATE)
2.数据类型以及是否为空等列属性不支持修改
3.一张表最多300列,且越少越好

### Kudu分片的限制
1.分片只支持手动指定,自动分片不支持
2.分片设定不支持修改,修改分片设定需要“新建表-导数据,删老表”
3.总切片数量不适宜过多,单个物理机最多承受1000个切片
4.表创建后，分区不能拆分或合并


> Kudu 还不支持跨多个 tablets 的事务
在编码或压缩之前，单元格不能大于 64KB 。
不支持关系功能，如外键。