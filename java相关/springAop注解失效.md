## springAop注解失效 2017-11-08

### 失效原因

```
    public String getCacheValue( String name ) {
        String cacheValue =getValue( name );
        return returnName;

    }
    @Cacheable( value = "dailyCache", key = "#name" )
    public String getValue( String name ) {
        String value="";
        //真正业务代码
        return vale;
    }

    //getCacheValue调用getValue()的时候，@Cacheable是不起作用的。 
    //@Cacheable是基于Spring AOP代理类,这样调用没有走代理。
```



### 解决办法

#### 方法1

* 两个方法写在不同的类中


####  方法2

1. 在xml配置aop代理
```
<!-- 避免aop 同一个类中 方法调用失效 -->
<aop:aspectj-autoproxy expose-proxy="true"/>

```

2. 方法调用加入当前类的代理

```
    public String getCacheValue( String name ) {
        
        //加入代理 xxxx为该类的名称
        String cacheValue =((xxxx)AopContext.currentProxy()).getValue( name );
        return returnName;

    }

    @Cacheable( value = "dailyCache", key = "#name" )
    public String getValue( String name ) {
        String value="";
        //真正业务代码
        return vale;
    }
```

