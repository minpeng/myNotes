## spring boot webScoket使用

### 1.pom文件配置
```
 <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-websocket</artifactId>
 </dependency>
```

### 2.webSocket bean配置

```
@Configuration
@EnableWebSocket
public class CafeWebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        //websocket 切点url
        registry.addHandler(CafeWebSocketHandler(), "/cafeWebSocket")
                .addInterceptors(CafeHandshakeInterceptor());
        //如果有跨域的情况,需要加入
        setAllowedOrigins("*");
    }
    
    //注入处理逻辑
    @Bean
    public WebSocketHandler CafeWebSocketHandler() {
        return new CafeWebSocketHandler();
    }
    
    //注入拦截器
    @Bean
    public HandshakeInterceptor CafeHandshakeInterceptor() {
        return new CafeHandshakeInterceptor();
    }
}
```

### 3.webSocket拦截器配置
```
//实现HandshakeInterceptor接口
public class CafeHandshakeInterceptor implements HandshakeInterceptor {
    //权限url
    private static final String PERMISSION="/webSocketUrl";
    
    //握手之前调用
    //权限判断
    @Override
    public boolean beforeHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Map<String, Object> map) throws Exception {
        ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) serverHttpRequest;

        //获取用户
        boolean user = ShiroKit.isUser();
        if(!user){
            return false;
        }
        //获取权限
        //判断是否有webSocket权限
        boolean permisson = ShiroKit.hasPermission(PERMISSION);
        if(!permisson){
            return false;
        }
        //把账号信息放到webSocketSession中
        map.put(WEBSOCKET_ACCOUNT,ShiroKit.getUser().getAccount());
        return true;
    }
    //握手之后调用
    @Override
    public void afterHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Exception e) {
        logger.info(ShiroKit.getUser().getAccount()+",成功连接cafeWebSocket...");
    }
}
```


### 4.webSocket消息处理器
```
@Component
public class CafeWebSocketHandler extends TextWebSocketHandler {
    //全局session,可以存所有用户的session,但是不支持分布式
    private final static CopyOnWriteArraySet<WebSocketSession> sessions = new CopyOnWriteArraySet<>();
   
   //在WebSocket协商成功后调用，并且打开WebSocket连接准备使用
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        super.afterConnectionEstablished(session);
        sessions.add(session);

        logger.info("【WebSocket】有新的连接, sessionId:{},account:{},总数:{}",
                session.getId(), (String) session.getAttributes().get(WEBSOCKET_ACCOUNT), sessions.size());

    }

    //关闭webSocket连接
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        super.afterConnectionClosed(session, status);
        sessions.remove(session);
        logger.info("【WebSocket】连接断开, sessionId:{},userName:{},总数:{}",
                session.getId(), (String) session.getAttributes().get(WEBSOCKET_ACCOUNT), sessions.size());
    }

    //发送消息
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        super.handleTextMessage(session, message);
        session.sendMessage(message);
    } 
}
```

### 5.前端配置
```

getWebSocket = function () {
    var webSocket = null;
    var heartBeatMessage="HeartBeat";
    if('WebSocket' in window){
        if(window.location.protocol==="https:"){
            webSocket = new WebSocket('wss://'+window.location.host+'/webSocket');
        }else{
            webSocket = new WebSocket('ws://'+window.location.host+'/webSocket');
        }

    }else{
        console.log("该浏览器不支持WebSocket!!!");
    }

    webSocket.onopen = function (event) {
        heartCheck.start();
        console.log("建立连接");
      

    }

    webSocket.onclose = function (event) {
        console.log("断开连接");
    }

    webSocket.onmessage = function (event) {
        heartCheck.reset();
        if(event.data==''|| event.data==undefined ||
            event.data==heartBeatMessage){
            return;
        }
        //获取数据
        var jsObject = JSON.parse(event.data);
     
    }

    webSocket.onerror = function (event) {
        console.log("websocket通信发生错误!!!");
    }

    window.onbeforeunload = function (event) {
        webSocket.close();
    }

    //心跳检测
    var heartCheck={
        timeout: 58*1000,//58s
        timeoutObj: null,
        reset: function(){
            clearTimeout(this.timeoutObj);
            this.start();
        },
        start: function(){
            this.timeoutObj = setTimeout(function(){
                webSocket.send("HeartBeat");
            }, this.timeout)
        }
    }
};
```