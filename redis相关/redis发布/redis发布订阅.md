## redis 发布/订阅

> 完善webSocket分布式session共享 

### 1.设置消息监听器

```
@Component
public class RedisContainerBean {
  

    @Bean
    RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory,
                                            MessageListenerAdapter listenerAdapter) {

        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        //订阅主题
        container.addMessageListener(listenerAdapter, new PatternTopic("chat"));

        return container;
    }
    //注册消息接收者方法
    //"receiveMessage" 是该RedisReceiver类的一个方法
    @Bean
    MessageListenerAdapter listenerAdapter(RedisReceiver receiver) {
        return new MessageListenerAdapter(receiver, "receiveMessage");
    }

    @Bean
    RedisReceiver receiver(CountDownLatch latch) {
        return new RedisReceiver(latch);
    }

    @Bean
    CountDownLatch latch() {
        return new CountDownLatch(1);
    }

    @Bean
    StringRedisTemplate template(RedisConnectionFactory connectionFactory) {
        return new StringRedisTemplate(connectionFactory);
    }
}

```

### 3.消息接收者
```
public class RedisReceiver {

    private static final Logger LOGGER = LoggerFactory.getLogger(RedisReceiver.class);

    private CountDownLatch latch;

    @Autowired
    public RedisReceiver(CountDownLatch latch) {
        this.latch = latch;
    }
    //接收方法
    public void receiveMessage(String message) {
        LOGGER.info("Received <" + message + ">");
        latch.countDown();
    }
}
```

### 4.测试

```
@Autowired
StringRedisTemplate redisTemplate;
    
@Test
public void testPubSub(){
    //发布消息
    //订阅该主题的都能收到
    redisTemplate.convertAndSend("chat", "redis pub sub!");
}

```