## redis 

### redis 启动命令 

```
redis-server.exe redis.windows.conf

## 连接
redis-cli.exe -h 127.0.0.1 -p 6379

##验证密码
auth 123456
```

### redis 常用命令
```
## 远程连接
redis-cli -h 127.0.0.1 -p 6379 -a password

##查看key
KEYS abc*

##删除
del

##查看类型
type key 

## 查看list 长度
LLEN key

## 取出list
LRANGE key 0 3

## 获取hash所有的key和filed值
HGETALL key

## 获取hash key 值
HGET key filed


## 订阅消息
SUBSCRIBE redisChat

## 发布消息
PUBLISH redisChat "Redis is a great caching technique"
```



### maven 打包跳过测试
```
mvn install -Dmaven.test.skip=true
mvn clean package  -Dmaven.test.skip=true
```