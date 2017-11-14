# spring异步发送邮件

## spring 发送邮件

```
package com.pengm.util;

import java.util.Properties;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSenderImpl;


/**
 * 发送告警邮件
 * 
 * @author pengm
 *
 */
public class MailUtil {
    private final static Logger logger = LoggerFactory.getLogger( CustomerAdminController.class );
    /**
     * 
     * @param subject 主题
     * @param text 内容
     */
    public static void sendMail( String subject, String text ) {
        JavaMailSenderImpl senderImpl = new JavaMailSenderImpl();
        senderImpl.setDefaultEncoding( "utf-8" );
        // 设定mail server
        senderImpl.setHost( "xxx" );
        senderImpl.setPort( 111 );

        // 建立邮件消息
        SimpleMailMessage mailMessage = new SimpleMailMessage();
        // 设置多人收
        String[] users = new String[] { "xxx" };
        mailMessage.setTo( users );
        mailMessage.setFrom( "xx" );
        mailMessage.setSubject( subject );
        mailMessage.setText( text );

        senderImpl.setUsername( "xxx" );
        senderImpl.setPassword( "xxxx" );

        Properties prop = new Properties();
        prop.put( "mail.smtp.auth", "true" ); // 将这个参数设为true，让服务器进行认证,认证用户名和密码是否正确
        prop.put( "mail.smtp.timeout", "25000" );
        senderImpl.setJavaMailProperties( prop );
        // 发送邮件
        senderImpl.send( mailMessage );

        logger.warn( "邮件发送成功..." );

    }

}

```


# sping 配置 异步模式

1. spring-mvc.xml 配置:需要加入命名空间:

> xmlns:task="http://www.springframework.org/schema/task"
>
> http://www.springframework.org/schema/task 
> http://www.springframework.org/schema/task/spring-task-4.1.xsd

完整配置: 
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:task="http://www.springframework.org/schema/task"
    xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.1.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.1.xsd   
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.1.xsd
        http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.1.xsd">

    <!-- 启用spring mvc 注解 -->
    <context:annotation-config />
    <!-- 自动扫描该包，使SpringMVC认为包下用了@controller注解的类是控制器 -->
    <context:component-scan base-package="com.cn21" />
    <mvc:annotation-driven>
        <mvc:message-converters register-defaults="true">
            <!-- 将StringHttpMessageConverter的默认编码设为UTF-8 -->
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8" />
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
    
    <!-- 异步模式 -->
    <!--异步定义推荐方式  -->  
    <task:executor id="executor" pool-size="15" />  
    <task:scheduler id="scheduler" pool-size="30" />  
    <task:annotation-driven executor="executor" scheduler="scheduler" />  
    
    <!-- 启动SpringMVC的注解功能，完成请求和注解POJO的映射 -->
    <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" />
    <!-- <bean
        class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
        <property name="messageConverters">
            <list>
                <ref bean="mappingJacksonHttpMessageConverter" /> JSON转换器
            </list>
        </property>
    </bean> -->
    
    <mvc:resources location="/assets/" mapping="/assets/**"/>
    <!-- 定义跳转的文件的前后缀 ，视图模式配置 -->
    <bean
        class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 这里的配置我的理解是自动给后面action的方法return的字符串加上前缀和后缀，变成一个 可用的url地址 -->
        <property name="prefix" value="/WEB-INF/views/" />
        <property name="suffix" value=".jsp" />
    </bean>
    
    
</beans>


```

AsyncTaskTest:
```
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

import com.pengm.util.MailUtil;

@Component
public class AsyncTaskTest {

    @Async
    public void doSomething() throws Exception {
        Thread.sleep( 10000 );
        System.out.println( "begin doSomthing...." );
        doSome();
        Thread.sleep( 10000 );
        System.out.println( "end doSomthing...." );
    }

    public static void doSome() {
        for( int i = 0; i < 10; i++ ) {
            try {
                Thread.sleep( 2000 );
                System.out.println( i );
            }
            catch( InterruptedException e ) {
                e.printStackTrace();
            }
        }
    }

}

```

adminController:
```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping(value="/admin")
public class adminController {
        

    @Autowired
    AsyncTaskTest asyncTask;

    @RequestMapping( "/test" )
    @ResponseBody
    public String asynTest() {
        try {
            System.out.println( "我开始执行了！" );
            asyncTask.doSomething();
            System.out.println( "我执行结束了！" );
        }
        catch( Exception e ) {
            e.printStackTrace();
        }
        return "account/index";
    }


}

```

后台打印结果：
> 我开始执行了！
> 我执行结束了！
> begin doSomthing....
> 0
> 1
> 2
> 3
> 4
> 5
> 6
> 7
> 8
> 9
> end doSomthing....


## 注意:
- 异步方法和调用类不要在同一个类中
- 注解扫描时，要注意过滤，避免重复实例化，因为存在覆盖问题，@Async就失效了
