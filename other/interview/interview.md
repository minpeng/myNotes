## interview

### 0927美团

1.Object的方法有哪些


![image](https://note.youdao.com/yws/api/personal/file/WEBc274da25951430c61d7c65f8a0fc3276?method=download&shareKey=eeecc53bd1ae6dc004b26c51200911ce)

```
共13个
1.registerNatives()：对本地方法进行注册
2.getClass()：返回一个对象的运行时类
3.hashCode():返回该对象的哈希值
4.equals():指示某个其他对象是否与此对象相等
5.clone():clone返回的对象为浅拷贝
6.toString():返回该对象的字符串表示
7.notify():唤醒此对象监视器上等待的单个线程
8.notifyAll():唤醒此对象监视器上等待的所有线程
9.wait():导致当前的线程等待，直到其它线程调用此对象的notify()或notifyAll()
10.finalize():当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法

```

> 引申

```

1.wait和sleep的区别是，wait释放了锁，sleep仍然持有锁

2.notify和notifyAll区别
notify():

唤醒在等待该对象同步锁的线程(只唤醒一个,如果有多个在等待),注意的是在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由JVM确定唤醒哪个线程，而且不是按优先级。

调用任意对象的notify()方法则导致因调用该对象的 wait()方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。

notifyAll():

唤醒所有等待的线程,注意唤醒的是notify之前wait的线程,对于notify之后的wait线程是没有效果的

所谓唤醒线程，另一种解释可以说是将线程由等待池移动到锁池，notifyAll调用后，会将全部线程由等待池移到锁池，然后参与锁的竞争，竞争成功则继续执行，如果不成功则留在锁池等待锁被释放后再次参与竞争。而notify只会唤醒一个线程
notify可能会导致死锁

```
```
1.sleep：Thread类的方法，必须带一个时间参数。会让当前线程休眠进入阻塞状态并释放CPU（阿里面试题 Sleep释放CPU，wait 也会释放cpu，因为cpu资源太宝贵了，只有在线程running的时候，才会获取cpu片段），提供其他线程运行的机会且不考虑优先级，但如果有同步锁则sleep不会释放锁即其他线程无法获得同步锁  可通过调用interrupt()方法来唤醒休眠线程。

 

2.yield：让出CPU调度，Thread类的方法，类似sleep只是不能由用户指定暂停多长时间 ，并且yield()方法只能让同优先级的线程有执行的机会。 yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。调用yield方法只是一个建议，告诉线程调度器我的工作已经做的差不多了，可以让别的相同优先级的线程使用CPU了，没有任何机制保证采纳。

 

3.wait：Object类的方法(notify()、notifyAll()  也是Object对象)，必须放在循环体和同步代码块中，执行该方法的线程会释放锁，进入线程等待池中等待被再次唤醒(notify随机唤醒，notifyAll全部唤醒，线程结束自动唤醒)即放入锁池中竞争同步锁

 

4.join：一种特殊的wait，当前运行线程调用另一个线程的join方法，当前线程进入阻塞状态直到另一个线程运行结束等待该线程终止。 注意该方法也需要捕捉异常。

等待调用join方法的线程结束，再继续执行。如：t.join();//主要用于等待t线程运行结束，若无此句，main则会执行完毕，导致结果不可预测
```

2.线程
 + 启动线程方式
 ```
 1.继承Thread类，并复写run方法，创建该类对象，调用start方法开启线程。此方式没有返回值
 2.实现Runnable接口，复写run方法，创建Thread类对象，将Runnable子类对象传递给Thread类对象。调用start方法开启线程。此方法2较之方法1好，将线程对象和线程任务对象分离开。降低了耦合性，利于维护。此方式没有返回值。
 3.创建FutureTask对象，创建Callable子类对象，复写call(相当于run)方法，将其传递给FutureTask对象（相当于一个Runnable）。 创建Thread类对象，将FutureTask对象传递给Thread对象。调用start方法开启线程。这种方式可以获得线程执行完之后的返回值。该方法使用Runnable功能更加强大的一个子类.这个子类是具有返回值类型的任务方法
 4.线程池：提供了一个线程队列，队列中保存着所有等待状态的线程。避免了创建与销毁额外开销，提高了响应的速度
 ```

 > 引申
 + runnable和thread的区别以及run和start的区别

 ```
 
 thread:线程对象各自占有各自的资源，并不是同时完成统一任务
 runnable:线程同时完成了统一的任务，可以达到资源共享的目的
 
 start():
 
 1.启动一个线程，不能多次启动一个线程。
 2.用start()方法来启动线程，真正实现了多线程运行，这时无需等待run方法体代码执行完毕而直接继续执行下面的代码
 
 run():
 1.在本线程内调用run()方法，可以重复多次调用。
 run()方法只是类的一个普通方法而已，如果直接调用Run方法，程序中依然只有主线程这一个线程，其程序执行路径还是只有一条，还是要顺序执行，要等待run方法体执行完毕后才可继
 
 ```


 3.mybaits

 + mybatis 原理

![image](https://note.youdao.com/yws/api/personal/file/WEB6785e319dd5ecbbc74a47d848f69473d?method=download&shareKey=8a46532b9a4547a5b6730ed7c007523e)

![image](https://note.youdao.com/yws/api/personal/file/WEBb2369d373f3f7e7023d1308193bbfd9f?method=download&shareKey=04fc3040ce1ba757fbc92f72e0e1399b)

![image](https://note.youdao.com/yws/api/personal/file/WEB8d317d10944dd2b88a45958837653cdc?method=download&shareKey=a2d5e6e91ef8938707704cf290d0ddf9)





 ```
 mybatis应用程序通过SqlSessionFactoryBuilder从mybatis-config.xml配置文件（也可以用Java文件配置的方式，需要添加@Configuration）中构建出SqlSessionFactory（SqlSessionFactory是线程安全的）；

然后，SqlSessionFactory的实例直接开启一个SqlSession，再通过SqlSession实例获得Mapper对象并运行Mapper映射的SQL语句，完成对数据库的CRUD和事务提交，之后关闭SqlSession

 ```
 + pagehelper分页原理

 ```
 pageHelper会使用ThreadLocal获取到同一线程中的变量信息，各个线程之间的Threadlocal不会相互干扰，也就是Thread1中的ThreadLocal1之后获取到Tread1中的变量的信息，不会获取到Thread2中的信息
所以在多线程环境下，各个Threadlocal之间相互隔离，可以实现，不同thread使用不同的数据源或不同的Thread中执行不同的SQL语句
所以，PageHelper利用这一点通过拦截器获取到同一线程中的预编译好的SQL语句之后将SQL语句包装成具有分页功能的SQL语句，并将其再次赋值给下一步操作，所以实际执行的SQL语句就是有了分页功能的SQL语句

 ```

 + #和$

 ```
 #是将传入的值当做字符串的形式
 $是将传入的数据直接显示生成sql语句
 ```


4.Spring
 + Benfactory和FactoryBean

 ```
 1.BeanFactory
 BeanFactory，以Factory结尾，表示它是一个工厂类(接口)， 它负责生产和管理bean的一个工厂
 
 
 2.FactoryBean
 以Bean结尾，表示它是一个Bean
 Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑
 ```

 + Bean的作用域
 ```
 1.单例singleton
 2.prototype：每次从容器中调用 Bean 时，都会返回一个新的实例，即相当于执行 new XxxBean() 的实例化操作
 3.requset：每次 http 请求都会创建一个新的 Bean ， 仅用于 WebApplicationContext 环境
 4.session：同一个 http Session 共享一个 Bean ，不同的 http Session 使用不同的 Bean，仅用于 WebApplicationContext 环境。
 5.globalSession：同一个全局 Session 共享一个 bean, 用于 Porlet, 仅用于 WebApplication 环境
 
 ```
 + 事务传播级别
 ```
 	//如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。默认为这个
    int PROPAGATION_REQUIRED = 0;
    
    //支持当前事务，如果当前没有事务，就以非事务方式执行
    int PROPAGATION_SUPPORTS = 1;
    
    //使用当前的事务，如果当前没有事务，就抛出异常。
    int PROPAGATION_MANDATORY = 2;
    
	//新建事务，如果当前存在事务，把当前事务挂起。
    int PROPAGATION_REQUIRES_NEW = 3;
    
    //以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
    int PROPAGATION_NOT_SUPPORTED = 4;
    
    //以非事务方式执行，如果当前存在事务，则抛出异常。
    int PROPAGATION_NEVER = 5;
    
    //如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作
    int PROPAGATION_NESTED = 6;
 ```
 + 事务隔离级别
 ```
 脏读：读取到了别的事务回滚前的数据，例如B事务修改数据库X，在未提交前A事务读取了X的值，而B事务发生了回滚。
不可重复读：一个事务在两次读取同一个数据的值不一致。例如A事务读取X，在中间过程中B事务修改了X的值，事务A再次读取X时值发生了改变。
幻读：查询得到的数据条数发生了改变，例如A事务搜索数据时有10条数据，在这时B事务插入了一条数据，A事务再搜索时发现数据有11条了
 ```

 + 数据库隔离级别

 ```
 read-uncommitted：未提交读（脏读、不可重复读、幻读）
read-committed：已提交读（不可重复读、幻读），大多数主流数据库的默认事务等级，保证了一个事务不会读到另一个并行事务已修改但未提交的数据，避免了“脏读取”。
repeatable-read：可重复读（幻读），保证了一个事务不会修改已经由另一个事务读取但未提交（回滚）的数据。
serializable：串行化最严格的级别，事务串行执行，资源消耗最大

 ```

 + 同一个类方法调用事务
 + aop实现
 ```
 JDK动态代理和cglib动态代理。
 JDK动态代理通过反射来接收被代理的类，但是被代理的类必须实现接口，核心是InvocationHandler和Proxy类
 。cglib动态代理的类一般是没有实现接口的类，cglib是一个代码生成的类库，可以在运行时动态生成某个类的子类
 
 ```
 + 代理的本质

+ java类启动流程
```
编译:.java文件编译后生成.class字节码文件
加载：类加载器负责根据一个类的全限定名来读取此类的二进制字节流到JVM内部，并存储在运行时内存区的方法区，然后将其转换为一个与目标类型对应的java.lang.Class对象实例
连接：细分三步
验证：格式（class文件规范） 语义（final类是否有子类） 操作
准备：静态变量赋初值和内存空间，final修饰的内存空间直接赋原值，此处不是用户指定的初值。解析：符号引用转化为直接引用，分配地址
初始化:有父类先初始化父类，然后初始化自己；将static修饰代码执行一遍，如果是静态变量，则用用户指定值覆盖原有初值；如果是代码块，则执行一遍操作
初值；如果是代码块，则执行一遍操作


```
+ 反射
```
Java的反射就是利用上面第二步加载到jvm中的.class文件来进行操作的。.class文件中包含java类的所有信息，当你不知道某个类具体信息时，可以使用反射获取class，然后进行各种操作

```
5. reids
 + 缓存雪崩
 + 缓存击穿

6.算法
 + 二叉树
 + LRU



### 0930晓教育


1.b树和B+树
```
1.b+树的数据都集中在叶子节点。分支节点 只负责索引。  b树的分支节点也有数据 。 b+树的层高 会小于 B树 平均的Io次数会远大于 B+树

2.b+树更擅长范围查询。叶子节点 数据是按顺序放置的双向链表。  b树范围查询只能中序遍历。

3.索引节点没有数据。比较小。b树可以吧索引完全加载至内存中。

B+tree是一种多路平衡查询树，节点是天然有序的，非叶子节点包含多个元素，不保存数据，只用来索引，叶子节点包含完整数据和带有指向下一个节点的指针，形成一个有序链表，有助于范围和顺序查找。因为非叶子节点不保存数据，所以同样大小的磁盘页可以容纳更多的元素，同样能数据量的情况下，B+tree相比B-tree高度更低，因此查询时IO会更少。
B-tree不管叶子节点还是非叶子节点，都会保存数据，这样导致在非叶子节点中能保存的指针数量变少（有些资料也称为扇出），指针少的情况下要保存大量数据，只能增加树的高度，导致IO操作变多，查询性能变低；
Hash索引底层是基于哈希表，就是以key-value存储数据的结构，多个数据在存储关系上是没有任何顺序关系的。只适合等值查询，不适合范围查询，而且也无法利用索引完成排序，不支持联合索引的最左匹配原则，如果有大量重复键值的情况下，哈希索引效率会很低，因为存在哈希碰撞。
二叉树：树的高度不均匀，不能自平衡，查找效率跟数据有关（树的高度），并且IO代价高。
红黑树：树的高度随着数据量增加而增加，IO代价高
```

2.shiro

3.tcc

4.springCloud

5.redis各个数据类型用法
```


String：缓存、计数器、分布式锁等。
List：链表、队列、微博关注人时间轴列表等。
Hash：用户信息、Hash 表等。
Set：去重、赞、踩、共同好友等。
Zset：访问量排行榜、点击量排行榜等。
```

6.redis单线程
```
1. 纯内存访问
2. I/O多路复用、事件驱动模型
3. 单线程避免了线程切换和竞态产生的消耗
```

7.redisString类型和Hash类型对比
```
编程稍微复杂
String优点：编程简单,
String缺点:序列化操作，每次更新需要全部读取之后再全部写入

Hash优点:节省空间，支持Hashtable和Ziplist，比String节约空间5倍,可以部分更新
Hash缺点:
Hash类型可以更节省内存，
避免String Json序列化
Redis的Key承载了很多特性的，过期时间、LRU、Cluster节点信息、系统关键信息等，因此和String类型的key-value相比，
hash类型可更大限度地减少key的数量，从而节省内存空间的使用率
```

8.java中有哪些锁？以及他们的区别

```
--公平锁：是指多个线程按照申请锁的顺序来获取锁。

--非公平锁：是指多个线程获取所得顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请锁的线程优先获取锁，有可能会造成优先级反转或者饥饿现象。

对比：一般开发用的默认是非公平锁，有点在于吞吐量比公平锁要大

--可重入锁：又名递归锁，是指在同一个线程的外层方法获取锁的时候，在进入内层方法会自动获取锁。

优点：对于Synchronized而言也是一个可重入锁。可重入锁的好处是可一定程度的避免死锁

--独享锁：是指该锁一次只能被一个线程所持有。

--共享锁：是指该锁可被多个线程持有。

Synchronized是独享锁
```

### 1010平安

1. 网络模型
 + tcp/ip 四层模型
 ```
 网络接口层
 网络层
 传输层
 应用层
 ```
 + osi七层模型
 ```
 物理层
 数据链路层
 网络层
 传输层
 表现层
 会话层
 应用层
 ```
2. tcp和udp

```
tcp：面向有连接，可靠传输，
udp：无连接，不可靠传输
```
3. tcp粘包/拆包
```
TCP粘包就是指发送方发送的若干包数据到达接收方时粘成了一包，从接收缓冲区来看，后一包数据的头紧接着前一包数据的尾，出现粘包的原因是多方面的，可能是来自发送方，也可能是来自接收方
一个数据包中包含了发送端发送的两个数据包的信息，这种现象即为粘包

1、要发送的数据大于TCP发送缓冲区剩余空间大小，将会发生拆包。
2、待发送数据大于MSS（最大报文长度），TCP在传输前将进行拆包。
3、要发送的数据小于TCP发送缓冲区的大小，TCP将多次写入缓冲区的数据一次发送出去，将会发生粘包。
4、接收数据端的应用层没有及时读取接收缓冲区中的数据，将发生粘包。

解决
1、发送端给每个数据包添加包首部，首部中应该至少包含数据包的长度，这样接收端在接收到数据后，通过读取包首部的长度字段，便知道每一个数据包的实际长度了。
2、发送端将每个数据包封装为固定长度（不够的可以通过补0填充），这样接收端每次从接收缓冲区中读取固定长度的数据就自然而然的把每个数据包拆分开来。
3、可以在数据包之间设置边界，如添加特殊符号，这样，接收端通过这个边界就可以将不同的数据包拆分开。
等等

```

3. socket
```
1.socket
2.bind
3.listen
4.read
5.wrtie
5.close
```
4.webSocket
```
WebSocket是一种在单个TCP连接上进行全双工通信的协议
Websocket 通过HTTP/1.1 协议的101状态码进行握手
请求头Upgrade: websocket和Connection: Upgrade表示这个连接将要被转换为WebSocket连接
```
5.分布式事务

```
1.2pc
2.3pc
3.tcc

```
6.消息队列(重复消费)
> 如何保证幂等性

```
设置消息全局唯一id,保证mq幂等
业务接收方法去重保证幂等
```


7.java泛型

```
泛型编程中，在 编译时就能获取到具体的类型 了，这一点带来的好处是使用类型推导避免强制类型转换，和编译期的类型安全检查

Java泛型实现依赖于类型擦除。在运行期，泛型类将会被转换为原始类。因此说，Java泛型是不彻底的泛型，仅仅是编译期的泛型
```

8.泛型数组
```
由于数组必须进行运行期类型检查，泛型数组也是数组，也要进行运行期类型检查；而由于类型擦除，造成数组运行期类型检查不能正常进行，破坏了Java数组运行期类型检查的机制，故不能使用泛型数组
```
```
T extends Comparable<T>
T[] array=(T[])new Comparable[10];
```
9.java1.7和1.8
+ 1.7
```
1.二进制变量的表示,支持将整数类型用二进制来表示，用0b开头
2.Switch语句支持String类型
3.Try-with-resource语句关闭资源
4.Catch多个异常
```
+ 1.8
```
1.default关键字，在接口中可以通过使用default关键字编写方法体，实现类可以不用实现该方法，可以进行直接调用
2.Lambda 表达式，函数式编程
3.新的日期api
4.流
5.Optional
6.jvm：元空间（Metaspace）： 一个新的内存空间，完全移除了永久代（方法区）（PermGen）。


```

10.Stream 惰性和非惰性

```
像 filter 这样只描述 Stream，最终不产生新集合的方法叫作 惰性求值方法;而像 count 这样 最终会从 Stream 产生值的方法叫作 及早求值方法

如果返回值是 Stream， 那么是惰性求值;如果返回值是另一个值或为空，那么就是及早求值

```

11.mysql优化
```
建表字段优化
尽可能字段not nul
索引优化

```

12.设计模式


13.redis发布/订阅优点，缺点

14.aof 

15.线程池


### 10.23顺丰

1.mysql索引
```
B+树
```
2.java内存模型

+ 程序计数器
```
1.线程安全
2.当前线程所执行的字节码行号指示器
```
+ 虚拟机栈
```
1.线程安全
2.每个方法执行时都会创建一个栈桢来存储方法的的变量表、操作数栈、动态链接方法、返回值、返回地址等信息
3.栈桢:局部变量,引用对象类型,操作数栈,返回地址类型
```
+ 本地方法栈
```
1.线程安全
```
+ 方法区
```
1.线程不安全
2.存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据

```
+ 堆

```
1.线程不安全
2.存储对象
```

3. volatile
```
1.轻量级锁
2.可见性和防止指重排
3.lock指令（内存屏障：将对缓存的修改操作立即写入主存）

```

4.ehreka注册中心原理

> AP（可用性和分区容错性）
> zookper(cp一致性和分区容错性)
```
1.注册中心
2.服务消费者
3.服务提供者

```
> eurek工作流程
```
1.Eureka Server 启动成功，等待服务端注册。在启动过程中如果配置了集群，集群之间定时通过 Replicate 同步注册表，每个 Eureka Server 都存在独立完整的服务注册表信息

2.Eureka Client 启动时根据配置的 Eureka Server 地址去注册中心注册服务

3.Eureka Client 会每 30s 向 Eureka Server 发送一次心跳请求，证明客户端服务正常

4.当 Eureka Server 90s 内没有收到 Eureka Client 的心跳，注册中心则认为该节点失效，会注销该实例

5.单位时间内 Eureka Server 统计到有大量的 Eureka Client 没有上送心跳，则认为可能为网络异常，进入自我保护机制，不再剔除没有上送心跳的客户端

6.当 Eureka Client 心跳请求恢复正常之后，Eureka Server 自动退出自我保护模式

7.Eureka Client 定时全量或者增量从注册中心获取服务注册表，并且将获取到的信息缓存到本地

8.服务调用时，Eureka Client 会先从本地缓存找寻调取的服务。如果获取不到，先从注册中心刷新注册表，再同步到本地缓存

9.Eureka Client 获取到目标服务器信息，发起服务调用

10.Eureka Client 程序关闭时向 Eureka Server 发送取消请求，Eureka Server 将实例从注册表中删除
```
### 10.23yy

1.实时uv,pv统计
```
1.源数据是从kafka->Hdfs的日志文件(每小时一个文件)
2.sprak,hive从hdfs分析数据,最后到Hbase
3.从Hbase读取
```
2.kafka
> 分布式消息队列,生产者往队列里写消息，消费者从队列里取消息进行业务逻辑
```
topic,一个topic有多个分区partition
Producer
Consumer
```
3.es

4.redis

>分布式锁 sexnx

5.java内存模型

### 10.24 涂鸦一面

tcc 2pc 3pc
代理模式
设计模式
jvm
垃圾回收

### 10.26 涂鸦二面
1. Hbase+phoenix

2. 付款消息弹框流程

   ```
   付款消息实时弹框及语音提醒流程
   CopyOnWriteArraySet:它是线程安全的无序的集合，可以将它理解成线程安全的HashSet
   1.客户端和服务端建立webSocket长连接,服务端会对当前用户进行角色校验,同时把登陆用户CopyOnWriteArraySet<WebSocketSession>记录在内存中
   2有用户付款,付款消息入redis队列,发送消息
   2.1判断该弹框的用户是否在线
   2.2判断当前是否已经正在弹框(redis key判断)
   2.3从队列里面取出第一个元素,把redis是否可弹框置为false,发送消息
   2.4在CopyOnWriteArraySet<WebSocketSession>找出符合接收人的用户，发送消息
   3.客户端接收消息，弹框，调用获取语音接口，语音播报
   3.1等用户关闭弹框或者10秒之后自动关闭，调用获取下一条收款信息接口
   3.2继续走2.1步骤
   
   
   多机器的时候CopyOnWriteArraySet<WebSocketSession>需要采用redis:pub/sub
   1.两台机器同时订阅同一个topic
   2.用户付款,发布消息，使订阅主题的都能收到
   3.收到之后，走上面2.1
   
   
   pub/sub的坏处就是sub端收不到订阅前发布的消息,sub端掉线之后也会丢失数据
   ```

   

reids队列遇到的问题
rabbitMa遇到的问题


tcc 2pc 3pc
代理模式
设计模式
jvm
垃圾回收
无
Volatile 修饰的成员变量在每次被线程访问时，都强迫从共享内存中重读该成员变量的值。而且，当成员变量发生变化时，强迫线程将变化值回写到共享内存。
这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值

spring tomcat替换
spring 初始化方法
CommandLineRunner 
实现了 CommandLineRunner 接口的 Component 会在所有 Spring Beans 初始化完成之后， 在 SpringApplication.run() 执行之前完成

实现 ApplicationRunner 接口的
ApplicationRunner 则是包含了 ApplicationArguments 对象，可以帮助获得更丰富的项目信息

@PostConstruct

InitializingBean


init-method
给bean配置init-method属性，或者在xml配置文件中指定，或者指定注解 Bean 的 initMethod 属性


顺序：
@PostConstruct->InitializingBean->init-method->ApplicationRunner->CommandLineRunner

spring 配置文件bootstrap.yml和application.yml
加载顺序：先加载bootstrap.yml再加载application.yml
boostrap 由父 ApplicationContext 加载，比 applicaton 优先加载
boostrap 里面的属性不能被覆盖

for update 什么时候释放
需要有事务，commit或者rollback释放
隔离级别 
读已提交
读未提交
可重复读
串行
造成数据库：脏读,不可重复读,幻读
传播属性
不可重复读和可重复读的区别

线程池参数
拒绝策略
1:丢弃抛出异常AbortPolicy 
2:丢弃不抛出异常DiscardPolicy 
3:丢弃最旧的任务，加入队列DiscardOldestPolicy 
4:交给线程池调用所在的线程进行处理CallerRunsPolicy 
theadLocal

mybatis常用标签：
insert,select,update,foreach,if,trim,set,choose,when,otherwise
mybaits预编译sql，把特殊字符转义


留存率计算
锁的分类
静态类和springbean
spring bean的类型
线程池四种

锁的分类
公平锁/非公平锁
可重入锁
独享锁/共享锁
互斥锁/读写锁
乐观锁/悲观锁
分段锁
偏向锁/轻量级锁/重量级锁
自旋锁

java排查cpu过高
1.top 查看pid
2.ps -mp pid -o THREAD,tid,time 查看tid
3.tid转换16进制printf "%x\n" pid
4.jstack pid |grep 16进制pid -A 30

### 10.29 微众银行
1.

### 10.30 oppo

```

```