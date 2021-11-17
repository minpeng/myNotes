## SpringBoot 优雅停机

### 版本>=2.3
```yaml

# 开启优雅停止 Web 容器，默认为 IMMEDIATE：立即停止
server.shutdown=graceful
# 最大等待时间
spring.lifecycle.timeout-per-shutdown-phase=30s

```


### 版本<2.3 
```java
//todo 

```


### 项目初始化启动DispatcherServlet 
```yaml
# 默认为-1，启动时候不加载
spring.mvc.servlet.load-on-startup=1


# 设置为1之后 启动项目 会输出   
Initializing Spring DispatcherServlet 'dispatcherServlet'...

```