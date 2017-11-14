## spring+memcache配置缓存注解@cahceable等


#### 1.增加memcache pom文件
```
<dependency>
    <groupId>com.googlecode.xmemcached</groupId>
    <artifactId>xmemcached</artifactId>
    <version>2.0.0</version>
</dependency>
```


### 1.配置memcache xml

```
<!-- xmemcached 配置 -->
        <bean name="memcachedClient"
            class="net.rubyeye.xmemcached.utils.XMemcachedClientFactoryBean"
            destroy-method="shutdown">
            <property name="servers">
                <!-- ip地址 端口号-->
                <value>127.0.0.1:11211</value>
            </property>
            <!-- server's weights -->
            <property name="weights">
                <list>
                    <value>1</value>
                </list>
            </property>
            <!-- nio connection pool size -->
            <property name="connectionPoolSize" value="2"></property>
            <!-- Use binary protocol,default is TextCommandFactory -->
            <property name="commandFactory">
                <bean class="net.rubyeye.xmemcached.command.BinaryCommandFactory"></bean>
            </property>
            <!-- Distributed strategy -->
            <property name="sessionLocator">
                <bean class="net.rubyeye.xmemcached.impl.KetamaMemcachedSessionLocator"></bean>
            </property>
            <!-- Serializing transcoder -->
            <property name="transcoder">
                <bean class="net.rubyeye.xmemcached.transcoders.SerializingTranscoder" />
            </property>
            <!-- ByteBuffer allocator -->
            <property name="bufferAllocator">
                <bean class="net.rubyeye.xmemcached.buffer.SimpleBufferAllocator"></bean>
            </property>
            <!-- Failure mode -->
            <property name="failureMode" value="false" />
        </bean>
```


### 3.增加cache 命名空间
```
<beans xmlns="http://www.springframework.org/schema/beans">
    xmlns:cache="http://www.springframework.org/schema/cache"

xsi:schemaLocation="http://www.springframework.org/schema/cache
http://www.springframework.org/schema/cache/spring-cache-3.1.xsd"
>
    //省略... 
</beans>
```


### 4.编写MemcachedCache管理类

```
package com.xxx.cache;

import java.util.concurrent.TimeoutException;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.Cache;
import org.springframework.cache.support.SimpleValueWrapper;



import net.rubyeye.xmemcached.MemcachedClient;
import net.rubyeye.xmemcached.exception.MemcachedException;


public class MemcachedCache implements Cache {
    
    //这里要注入该项目的缓存
    @Autowired
    private MemcachedClient memcachedClient;

    private String name;

    public MemcachedCache() {
          
      }

    public MemcachedCache(String name, MemcachedClient memcachedClient) {
        this.memcachedClient = memcachedClient;
        this.name = name;  
      }

    @Override
    public String getName() {
        return this.name;
    }

    @Override
    public Object getNativeCache() {
        return this.memcachedClient;
    }

    @Override
    public ValueWrapper get( Object key ) {
        Object value = null;
        try {
            value = this.memcachedClient.get( objectToString( key ) );
        }
        catch( TimeoutException | InterruptedException | MemcachedException e ) {

            e.printStackTrace();
        }
        return (value != null ? new SimpleValueWrapper( value ) : null);
    }

    @Override
    public void put( Object key, Object value ) {
        try {
            this.memcachedClient.set( objectToString( key ), 3600, value );
        }
        catch( TimeoutException | InterruptedException | MemcachedException e ) {

           e.printStackTrace();
        }

    }

    @Override
    public void evict( Object key ) {
        try {
            this.memcachedClient.delete( objectToString( key ) );
        }
        catch( TimeoutException | InterruptedException | MemcachedException e ) {

            e.printStackTrace();
        }

    }

    @Override
    public void clear() {

    }

    private static String objectToString( Object object ) {
        if( object == null ) {
            return null;
        }
        else if( object instanceof String ) {
            return (String)object;
        }
        else {
            return object.toString();
        }
    }



    public void setName( String name ) {
        this.name = name;
    }

}


```


### 5.配置CacheManager
```
    <!-- 启动缓存注解功能 -->
    <cache:annotation-driven/>

    <bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
        <property name="caches">
            <set>
                <bean class="com.xxx.cache.MemcachedCache" p:name="xxxx"/>
            </set>
        </property>
    </bean>

```

#### 6.启用注解

```
    @Cacheable( value = "xxxx", key = "#name" )
    public String getName( String name ) {
        System.err.println( "执行该方法:" + new Date() );
        //do something
        return name;
    }

```