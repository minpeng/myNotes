## shiro 权限控制

### 1.增加jar包

```

    <!--shiro start -->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>${shiro.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-web</artifactId>
            <version>${shiro.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>${shiro.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>${shiro.version}</version>
        </dependency>

     <!--shiro end -->
```


### 2.配置xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd"
    default-lazy-init="false">

    <description>Shiro安全配置</description>

    <bean id="sessionDAO"
        class="org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO" />
    
    <bean id="sessionIdCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
        <property name="name" value="xxxxxx"></property>
        <property name="path" value="/"></property>
    </bean>
    
    <bean id="shiroSessionManager"
        class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
        <property name="sessionDAO" ref="sessionDAO" />
        <property name="sessionValidationInterval" value="1800000" />
        <property name="globalSessionTimeout" value="1800000" />
        <property name="sessionIdUrlRewritingEnabled" value="false" />
        <property name="sessionIdCookieEnabled" value="true" />
        <property name="sessionIdCookie" ref="sessionIdCookie" />

    </bean>
    
    
    <!-- Shiro's main business-tier object for web-enabled applications -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <!--缓存配置-->
        <property name="cacheManager" ref="shiroCacheManager" />
        <property name="sessionManager" ref="shiroSessionManager" />
        <property name="realms">
            <list>
                <ref local="shiroHttpRealm" />
            </list>
        </property>
    </bean>
    
    <!-- 编写域 -->
    <bean id="shiroHttpRealm" class="xxx.xx.ShiroHttpRealm">
    </bean>

    <!-- Shiro Filter -->
    <bean id="roleOrFilter"
        class="xxx.CustomRolesAuthorizationFilter" />
    <bean id="updateCurrentInfoFilter" 
        class="xxx.ModuleAuthorizationFilter" />
    <bean id="validateFilter" 
        class="xxx.ValidateURLFilter" />
    <bean id="productFilter" 
        class="xxx.ProductAuthFilter" />
    <bean id="permissionInfoFilter" 
        class="xxx.PermissionInfoFilter" />
        
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="securityManager" ref="securityManager" />
        <property name="loginUrl" value="${APP_URL}/login" />
        <property name="successUrl" value="/index" />
        <property name="unauthorizedUrl" value="/unauthorized" />
        <property name="filters">
            <map>
                <entry key="roles" value-ref="roleOrFilter" />
                <entry key="current" value-ref="updateCurrentInfoFilter" />
                <entry key="validate" value-ref="validateFilter" />
                <entry key="productAuth" value-ref="productFilter" />
                <entry key="permissionInfo" value-ref="permissionInfoFilter" />
            </map>
        </property>

        <!-- 从上到下，从左往右验证-->
        <property name="filterChainDefinitions">
            <value>
                <!-- anon 任何用户发送的请求都能够访问-->
                /unauthorized = anon
                /assets/** = anon
                /login/** = anon

                 <!-- user 用户是身份验证通过或RememberMe 登录-->
                /feedback/** = user
               
                 <!-- authc 登录成功才能够访问-->
                /common/**=authc
               
               <!-- 具有1,2,3角色,通过productAuth,validate,permissionInfo过滤器验证-->
                /**/edit**=roles[1,2,3],productAuth,validate,permissionInfo
                
             <!-- 具有1角色,通过permissionInfo过滤器验证-->
                /adminSettings/**=roles[1],permissionInfo
                
                /index/**=authc
                /** = authc
                
            </value>
        </property>
    </bean>


    <!-- 保证实现了Shiro内部lifecycle函数的bean执行 -->
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
    
</beans>

```


### 3.编写shiro域

```
public class ShiroHttpRealm extends AuthorizingRealm {
    
    //判断用户是否正常，正常登录返回该用户信息
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo( PrincipalCollection principals ) {
        // TODO Auto-generated method stub
        return null;
    }
    
    //加载该用户权限信息，方便后面做判断
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo( AuthenticationToken token ) throws AuthenticationException {
        // TODO Auto-generated method stub
        return null;
    }

    


}

```

#### 4.filters
```
//继承AuthorizationFilter
public class ProductAuthFilter extends AuthorizationFilter {


    @Override
    protected boolean isAccessAllowed( ServletRequest request, ServletResponse response, Object mappedValue ) throws Exception {

        Subject subject = getSubject( request, response );
        if( subject == null ) {
            return false;
        }

        ShiroUser user = (ShiroUser)subject.getPrincipal();
        long userId = user.id;

        if( userId < 0L ) {
            return false;
        }
        // 获取请求的产品
        String productId = request.getParameter( "productId" );
        if( StringUtils.isEmpty( productId ) ) {
            return true;
        }
        //具体业务
        return true;

    }


```

#### 5.缓存配置

- shiro 缓存
```
import org.apache.shiro.cache.AbstractCacheManager;
import org.apache.shiro.cache.Cache;
import org.apache.shiro.cache.CacheException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component( value = "shiroCacheManager" )
public class ShiroCacheManager extends AbstractCacheManager{

    @Autowired
    private ShiroCache myCache;
    
    @Override
    protected Cache createCache(String arg0) throws CacheException {
        return myCache;
    }

}


```

- 存储于memcache
```

import java.util.Collection;
import java.util.Set;

import org.apache.shiro.cache.Cache;
import org.apache.shiro.cache.CacheException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import net.rubyeye.xmemcached.MemcachedClient;

@Component
public class ShiroCache implements Cache<Object, Object> {

    public static final int HOUR_TIME_OUT = 60 * 60 * 1;

    @Autowired
    private MemcachedClient memcachedClient;
    
    public Object get( Object key ) throws CacheException {
        if( key instanceof String ) {
            try {
                return this.memcachedClient.get( (String)key );
            }
            catch( Exception exception ) {
                exception.printStackTrace();
                return null;
            }
        }
        return null;
    }

    @Override
    public Object put( Object key, Object value ) throws CacheException {
        if( key instanceof String ) {
            try {
                this.memcachedClient.set( (String)key, HOUR_TIME_OUT, value );
            }
            catch( Exception e ) {
                e.printStackTrace();
                return null;
            }
        }
        return value;
    }
    
    @Override
    public Set<Object> keys() {
        // TODO Auto-generated method stub
        return null;
    }
    
    @Override
    public Object remove(Object arg0) throws CacheException {
        // TODO Auto-generated method stub
        return null;
    }

    @Override
    public int size() {
        // TODO Auto-generated method stub
        return 0;
    }

    @Override
    public Collection<Object> values() {
        // TODO Auto-generated method stub
        return null;
    }
    
    @Override
    public void clear() throws CacheException {
        // TODO Auto-generated method stub
        
    }
```


### springboot shiro拦截器

``` 
        Map<String, String> hashMap = new HashMap<>();
        hashMap.put("/settlement/**", "permissionUrlFilter");
        shiroFilter.setFilterChainDefinitionMap(hashMap);
        //自定义拦截器
        Map<String, Filter> filters=new HashMap<>();
        filters.put("permissionUrlFilter",permissionUrlFilter());
        shiroFilter.setFilters(filters);


        @Bean
        public PermissionUrlFilter permissionUrlFilter() {
            return new PermissionUrlFilter();
        }
```