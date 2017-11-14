### quartz实现job接口但是无法注入的问题


#### 解决quartz的job中使用autowired注解注入的对象为空。

1.继承AdaptableJobFactory类
```

public class JobFactory extends AdaptableJobFactory {  
      
    @Autowired  
    private AutowireCapableBeanFactory capableBeanFactory;  
  
    @Override  
    protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {  
        //调用父类的方法  
        Object jobInstance = super.createJobInstance(bundle);  
        //进行注入  
        capableBeanFactory.autowireBean(jobInstance);  
        return jobInstance;  
    }  
      
}  
```

2.修改xml

```
<bean id="jobFactory" class="com.xxx.xxx.JobFactory"></bean>
    <bean id="schedulerFactoryBean"  autowire="no" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">  
         　<!-- 其他属性省略 -->
         <property name="jobFactory" ref="jobFactory">
         </property>  
    </bean>  
```