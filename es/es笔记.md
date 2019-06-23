## es笔记

1.基本属性
```
_index:索引(数据库database)
_type:类型(数据库的表table)
_id:唯一id(数据库主键id)

mapping:映射(数据库表的建表语句或者表示表字段)

document:文档(数据库表的一行数据)

_source:查询返回的document


```

2.基本操作

```
1.查询
GET /_index/_type/_id

2.检索文档是否存在
curl -i -XHEAD http://localhost:9200/_index/_type/_id

3.创建 
POST /_index/_type/
{ ... }

4.删除
DELETE /_index/_type/_id


5.文档更新 
POST /_index/_type/_id/_update
对象合并在一起，存在的标量字段被覆盖，新字段被添加
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}

6.更新可能不存在的文档
upsert参数定义文档来使其不存在时被创建
POST /index/_type/_id/_update
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}

```

3.搜索

```
1.空搜索(返回全部数据)
GET /_search

返回字段：
took:搜索请求花费的毫秒数
_score:相关性得分(relevance score),文档与查询的匹配程度
_shards:节查询的分片数（total字段），成功的（successful字段），失败的（failed字段）
time_out:是否超时


/_index1/_search 限定搜索_index1

/_index1,_index2/_search 多个索引搜索

/g*,u*/_search 以g或者u开头的索引搜索

```
