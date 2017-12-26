## 查询mysql各个表的数据量


```

##查询出来的是每张表的行数

use information_schema;
select table_name,table_rows from tables
where TABLE_SCHEMA = '数据库名'
order by table_rows desc;

```