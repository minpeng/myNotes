### Mysql Order By 排序优化

t1 有两个索引
key1(key1_name,key1_age),key2(key2_id)

>不能利用索引排序

```

//1.排序字段在多个索引中，无法使用索引排序
SELECT * FROM t1 ORDER BY key1_name,key1_age, key2_id;
 
//2.排序键顺序与索引中列顺序不一致，无法使用索引排序
SELECT * FROM t1 ORDER BY key2_id, key1_name;
 
//3.升降序不一致，无法使用索引排序
SELECT * FROM t1 ORDER BY key1_name DESC, key2_id ASC;
 
//4.key1_age是范围查询，key_part2无法使用索引排序
SELECT * FROM t1 WHERE key1_age> constant ORDER BY key2_id;

```

>利用索引排序

```
//1.索引顺序一致
SELECT * FROM t1 ORDER BY key1_name,key1_age;

//2.
SELECT * FROM t1 WHERE key1_name = constant ORDER BY key1_age;

//3.范围查询和排序字段一致
SELECT * FROM t1 WHERE key1_name > constant ORDER BY key1_name ASC;

//4.
SELECT * FROM t1 WHERE key1_name = constant1 AND key1_age > constant2 ORDER BY key1_age;
```