## SpringBoot自定义配置文件区分环境配置


> 自定义的配置文件也要区分环境

### 1.设置自定义的配置文件

+  my-dev.properties

```

##只写一个属性方便测试

my.name=dev

```


+ my-test.properties

```

##只写一个属性方便测试
my.name=test

```


### 2.编写配置文件实体类
```JAVA

/**
 * 自定义配置文件属性
 */
public class MyConfig {
   
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public MyConfig(String name) {
        this.name = name;
    }
}
```


### 2.编写配置文件接口

> 方便spring注入,也可以增加业务逻辑

```JAVA
/**
 * 配置文件接口,方便注入或者做一些其他业务逻辑
 */
public interface IMyConfig {
    //获取配置文件属性
    MyConfig getMyConfig();
}


```
### 3.编写不同环境下对应的实体类

+ test
```JAVA
/**
 * test环境配置
 */
@Component("myConfig")
@Configuration
//文件名
@PropertySource("classpath:my-test.properties")
//前缀
@ConfigurationProperties(prefix="my")
//test环境
@Profile("test")
public class MyConfigTest implements IMyConfig{
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
    @Override
    public MyConfig getMyConfig() {
        return new MyConfig(this.name);
    }
}    

```

+ dev

```JAVA
/**
 * dev环境配置
 */
@Component("myConfig")
@Configuration
//文件名
@PropertySource("classpath:my-dev.properties")
//前缀
@ConfigurationProperties(prefix="my")
//dev环境
@Profile("dev")
public class MyConfigDev implements IMyConfig{
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public MyConfig getMyConfig() {
        return new MyConfig(this.name);
    }
}


```
### 4.编写测试类验证

```JAVA

@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTests {
	@Autowired
	IMyConfig myConfig;
	@Test
	public void testMyConfig() {
		MyConfig myConfig = this.myConfig.getMyConfig();
		//修改不同环境可以得到不同的值
		System.out.println(myConfig.getName());
	}

}

```