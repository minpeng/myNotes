## spring redis 配置

1.appliaction.xml配置
```
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=

```

2.修改序列化

```
@Autowired
RedisTemplate redisTemplate;

@Autowired
public void setRedisTemplate(RedisTemplate redisTemplate) {
    RedisSerializer stringSerializer = new StringRedisSerializer();
    Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
    ObjectMapper om = new ObjectMapper();
    // 设置可见度
    om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
    // 启动默认的类型
    om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
    // 序列化类，对象映射设置
    jackson2JsonRedisSerializer.setObjectMapper(om);

    redisTemplate.setKeySerializer(stringSerializer);
    redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
    redisTemplate.setHashKeySerializer(stringSerializer);
    redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

    this.redisTemplate = redisTemplate;
}

public Object getObjectValue(String key) {
    return redisTemplate.opsForValue().get(key);
}

public void setObjectCache(String key, Object value, long l) {
    redisTemplate.opsForValue().set(key, value, l, TimeUnit.SECONDS);
}

public void removeCache(String key) {
    redisTemplate.delete(key);
}
```