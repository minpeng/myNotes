## okhttp使用


### 动态超时时间配置
> 新增一个拦截器,在拦截器里面动态调整

```java

public class DynamicConnectTimeout implements Interceptor {

    /**
     * 超时时间配置
     * keu：域名
     * value：超时时间-单位毫秒
     */
    private static final Map<String, Integer> URL_MAP = ImmutableMap.of(
            "baidu", 200,
            "163", 300
    );

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        //获取url
        String questUrl = request.url().toString();

        for (Map.Entry<String, Integer> entry : URL_MAP.entrySet()) {
            String key = entry.getKey();
            if (questUrl.contains(key)) {
                Integer timeOut = entry.getValue();
                return chain.withConnectTimeout(timeOut, TimeUnit.MILLISECONDS)
                        .withReadTimeout(timeOut, TimeUnit.MILLISECONDS)
                        .withWriteTimeout(timeOut, TimeUnit.MILLISECONDS)
                        .proceed(request);
            }
        }
        return chain.proceed(request);

    }
}

```


### 耗时监控
> 继承EventListener,重写方法，增加耗时统计
```java

@Slf4j
public class MyEventListener extends EventListener {
    public MyEventListener() {
        super();
    }

    @Override
    public void callStart(Call call) {
        log.info("开始请求 callStart...");
        super.callStart(call);
    }

    @Override
    public void dnsStart(Call call, String domainName) {
        log.info("开始解析dns dnsStart...");
        super.dnsStart(call, domainName);
    }

    @Override
    public void dnsEnd(Call call, String domainName, List<InetAddress> inetAddressList) {
        log.info("结束解析dns dnsEnd...");
        super.dnsEnd(call, domainName, inetAddressList);
    }

    //OkHttp是使用Socket接口建立Tcp连接的，所以这里的连接就是指Socket建立一个连接的过程。
    //当连接被重用时，connectStart/connectEnd不会被调用。
    //当请求被重定向到新的域名后，connectStart/connectEnd会被调用多次。
    @Override
    public void connectStart(Call call, InetSocketAddress inetSocketAddress, Proxy proxy) {
        log.info("连接开始 connectStart");
        super.connectStart(call, inetSocketAddress, proxy);
    }

    //secureConnectStart/secureConnectEnd TLS安全连接开始和结束
    //如果我们使用了HTTPS安全连接，在TCP连接成功后需要进行TLS安全协议通信，等TLS通讯结束后才能算是整个连接过程的结束，也就是说connectEnd在secureConnectEnd之后调用。
    //当存在重定向或连接重试的情况下，secureConnectStart/secureConnectEnd会被调用多次。
    //
    //在Socket建立连接后，会执行一个establishProtocol方法，这个方法的作用就是TSL/SSL握手。
    @Override
    public void secureConnectStart(Call call) {
        log.info("ssl连接开始 secureConnectStart");
        super.secureConnectStart(call);
    }

    @Override
    public void secureConnectEnd(Call call, Handshake handshake) {
        log.info("ssl连接结束 secureConnectStart");
        super.secureConnectEnd(call, handshake);
    }

    @Override
    public void connectEnd(Call call, InetSocketAddress inetSocketAddress, Proxy proxy, Protocol protocol) {
        log.info("连接结束 connectEnd");
        super.connectEnd(call, inetSocketAddress, proxy, protocol);
    }

    @Override
    public void connectFailed(Call call, InetSocketAddress inetSocketAddress, Proxy proxy, Protocol protocol, IOException ioe) {
        log.info("连接失败 connectFailed");
        super.connectFailed(call, inetSocketAddress, proxy, protocol, ioe);
    }


    //connectionAcquired/connectReleased 连接绑定和释放
    //因为OkHttp是基于连接复用的，当一次请求结束后并不会马上关闭当前连接，而是放到连接池中，当有相同域名的请求时，会从连接池中取出对应的连接使用，减少了连接的频繁创建和销毁。
    //当根据一个请求从连接池取连接时，并打开输入输出流就是acquired，用完释放流就是released。
    //
    //connectionAcquired是在连接成功后被调用的。但是在连接复用的情况下没有连接步骤，connectAcquired会在获取缓存连接后被调用。由于StreamAllocation是连接“Stream”和“Connection”的桥梁，所以在StreamAllocation中会持有一个RealConnection引用。StreamAllocation在查找可用连接的顺序为：StreamAllocation.RealConnection -> ConnectionPool -> ConnectionPool -> new RealConnection
    @Override
    public void connectionAcquired(Call call, Connection connection) {
        super.connectionAcquired(call, connection);
    }

    @Override
    public void connectionReleased(Call call, Connection connection) {
        super.connectionReleased(call, connection);
    }

    @Override
    public void requestHeadersStart(Call call) {
        super.requestHeadersStart(call);
    }

    @Override
    public void requestHeadersEnd(Call call, Request request) {
        super.requestHeadersEnd(call, request);
    }

    @Override
    public void requestBodyStart(Call call) {
        super.requestBodyStart(call);
    }

    @Override
    public void requestBodyEnd(Call call, long byteCount) {
        super.requestBodyEnd(call, byteCount);
    }

    @Override
    public void responseHeadersStart(Call call) {
        super.responseHeadersStart(call);
    }

    @Override
    public void responseHeadersEnd(Call call, Response response) {
        super.responseHeadersEnd(call, response);
    }

    @Override
    public void responseBodyStart(Call call) {
        super.responseBodyStart(call);
    }

    @Override
    public void responseBodyEnd(Call call, long byteCount) {
        super.responseBodyEnd(call, byteCount);
    }

    @Override
    public void callEnd(Call call) {
        log.info("结束请求 callEnd...");
        super.callEnd(call);
    }

    @Override
    public void callFailed(Call call, IOException ioe) {
        log.info("请求有错误 callFailed...");
        super.callFailed(call, ioe);
    }
}
```

