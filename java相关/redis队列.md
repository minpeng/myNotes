
## redis 实现队列


```
@Component("redisQueue")
public class RedisQueue {
    @Autowired
    private RedisTemplate redisTemplate;
    
    //设置序列化方式
    @Autowired
    public void setRedisTemplate(RedisTemplate redisTemplate) {
        RedisSerializer stringSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(stringSerializer);
        redisTemplate.setValueSerializer(stringSerializer);
        redisTemplate.setHashKeySerializer(stringSerializer);
        redisTemplate.setHashValueSerializer(stringSerializer);
        this.redisTemplate = redisTemplate;
    }

    /**
     * 列表添加
     *
     * @param key
     * @param value
     */
    public void lPush(String key, Object value) {
        ListOperations<String, Object> list = redisTemplate.opsForList();
        list.rightPush(key, value);
    }

    /**
     * 队列获取
     *
     * @param key
     * @param start
     * @param end
     * @return
     */
    public List<Object> lRange(String key, long start, long end) {
        ListOperations<String, Object> list = redisTemplate.opsForList();
        return list.range(key, start, end);
    }

    /**
     * 取出第一个并且删除
     *
     * @param key
     */
    public Object lPop(String key) {
        ListOperations<String, Object> list = redisTemplate.opsForList();
        Object value=list.leftPop(key);
        return value;

    }

    /**
     * 队列删除
     *
     * @param key
     * @param start
     * @param end
     * @return
     */
    public void lDelete(String key, long start, long end) {
        ListOperations<String, Object> list = redisTemplate.opsForList();
        list.trim(key, start, end);
    }

    public void lDeleteAll(String key) {
        lDelete(key, 1, 0);
    }
}

```