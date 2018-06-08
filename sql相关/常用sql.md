## 常用sql


```
# 查看数据库正在执行的sql
SHOW   PROCESSLIST;
```


```
//增加字段
ALTER TABLE t_bigdata_operation_retention ADD gameName VARCHAR(100) NOT NULL DEFAULT "ALL" AFTER appkey;

```

```
##修改字段长度
ALTER TABLE t_bigdata_product_config MODIFY COLUMN  CATE_NAME VARCHAR(50);
```