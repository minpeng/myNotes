## 使用mybatis order by排序失效

> 占位符应该使用$而不是#

```

1.order by #{id},会把传入的参数加一个双引号,变成order by "id"
2.order by ${id},$将传入的数据直接显示,order by id

```