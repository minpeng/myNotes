## redis 使用lua脚本

```java

//初始化
 private static DefaultRedisScript<Long> redisScript;

    static {
        redisScript = new DefaultRedisScript<>();
        redisScript.setResultType(Long.class);
        redisScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("scripts/lua/test.lua")));
    }


//使用
List<String> keys = new ArrayList<>();
    keys.add(key);
    Long result = redisTemplate.execute(redisScript, keys, String.valueOf(productTime));


```