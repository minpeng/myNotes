## 微服务基础

1.网关

```
zuul 中定义了四种不同生命周期的过滤器类型：
     
1、pre：可以在请求被路由之前调用;
2、route：在路由请求时候被调用;
3、post：在route和error过滤器之后被调用;
4、error：处理请求时发生错误时被调用;

```


2.Security

```
1.AuthenticationProvider接口:实现该接口来自定义设置认证逻辑
2.AccessDecisionManager:用户访问控制,实现该接口可以对用户是否可以访问某个权限进行设定
3.AccessDecisionVoter：投票器,在授权的时通过投票的方式来决定用户是否可以访问
4.UserDetailsService:加载特定的用户信息
```

3.gw转发修改参数
```
 // 获取body中的请求参数
String requestBody = null;
try {
    requestBody = IOUtils.toString(request.getReader());
} catch (IOException e) {
}

// 转化成json
JSONObject json = new JSONObject();
if (StringUtils.isNotEmpty(requestBody)) {
    json = JSONObject.parseObject(requestBody);
}
json.put(key, value);
//加入这个可以直接@RequestBody获取
json.putAll(value.toMap());
String newBody = json.toString();

final byte[] newBodyBytes = newBody.getBytes();
requestContext.setRequest(new HttpServletRequestWrapper(request) {
    @Override
    public ServletInputStream getInputStream() throws IOException {
        return new ServletInputStreamWrapper(newBodyBytes);
    }

    @Override
    public int getContentLength() {
        return newBodyBytes.length;
    }

    @Override
    public long getContentLengthLong() {
        return newBodyBytes.length;
    }

});
```

微服务公共模块打包
```
<build>
    <plugins>
        <!--打包jar-->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
        </plugin>

        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <skip>true</skip>
            </configuration>
        </plugin>
    </plugins>

    </build>
```