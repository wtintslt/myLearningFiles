
# 基础

>- 数据类型

| 类型 		| 范围 					| 二进制位数	| 默认值	|
| --		| -- 					| -- 		| 	--	|
| boolean 	| true/false			| 1 		| false |
| char 		| \u0000 ~ \uffff 		| 16 		|'u0000'|
| byte 		| -2^7 ~ 2^7-1 			| 8 		| 	0 	|
| short 	| -2^15 ~ 2^15-1 		| 16 		| 	0 	|
| int 		| -2^31 ~ 2^31-1 		| 32 		| 	0 	|
| long 		| -2^63 ~ 2^63-1		| 64 		| 	0L 	|
| float 	| 1.4E-45 ~ 3.4028235E38| 32 		| 	0.0f|
| double 	| 4.9E-324 ~ 1.7976931348623157E308| 64| 0.0d|

 **自动转换的规则是从低级类型数据转换成高级类型数据**。  
转换规则如下:

数值型数据的转换：byte→short→int→long→float→double (float 只存6位小数 + 数据的位数)  
字符型转换为整型：char→int  


+ 当基本类型声明在对象中，作为属性出现，则其作为成员变量，内存在**堆空间**中分布
+ 当基本类型作为局部变量，即方法体中出现，则内存分配在**栈**中


  ## 设计原则
 >- 
 - 单一职责： 一个类只负责一个职责
 - 迪米特原则：一个类对自己耦合或者知道的类越少越好
 - 里氏替换：能用父类的地方就能用子类替换
 - 依赖倒置：实现类之间不发生直接的依赖关系，依赖关系通过接口或抽象类产生
 - 接口隔离：使用多个专门的接口比使用单个接口好；接口单一，方法尽量少
 - 开闭原则：对扩展开放，对修改关闭

---
# java内存

## happens-before原则
+ 程序次序规则：在一个线程内，按照控制流顺序，书写在前面的操作先行发生于书写在后面的操作
+ 锁定规则：一个unlock操作先行发生于后面对同一个锁lock操作
+ volatile原则：对一个变量的写操作先行发生于对这个变量的读操作
+ 传递规则：操作A先行发生于操作B，B先行发生于C，那么A先行发生于C
+ 线程启动规则：Thread对象的start()方法先行发生于这个线程的每一个动作
+ 线程终止规则：线程的所有操作都线程发生于对此线程的终止检测，可以通过Thread::join()方法是否结束、Thread::isAlive()的返回值等手段检测线程是否已经终止执行
+ 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过Thread::interrupted()方法检测到是否有中断发生
+ 对象终结规则：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始
+ 传递性原则：如果操作A先行发生于B，操作B先行发生于C，那么操作A先行发生于操作C

# 框架
## spring

1、springboot 启动流程
(a)加载启动类：SpringBoot通过反射机制加载启动类，该类必须带有@SpringBootApplication注解。
(b)启动Spring容器：启动类中的main方法会调用SpringApplication类的静态方法run，该方法会创建一个SpringApplication实例，并启动Spring容器。
(c)加载配置文件：SpringApplication会读取classpath中的配置文件，包括application.properties和application.yml等，并将其转换为Spring Boot应用程序中的属性。
(d)注册Bean定义：SpringBoot会扫描启动类所在包及其子包中的所有类，并将其转换为Spring Bean，以便后续注入和管理。
(e)启动应用程序：Spring Boot会调用Spring容器中所有实现了CommandLineRunner接口的Bean的run方法，以便在应用程序启动后执行一些初始化操作。

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    // 获取Spring应用的初始化器
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 通知Spring应用的初始化器开始启动
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        // 通过Spring应用上下文初始化器创建Spring应用上下文
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        configureIgnoreBeanInfo(environment);
        Banner printedBanner = printBanner(environment);
        // 创建Spring应用上下文
        context = createApplicationContext();
        // 通过Spring应用上下文初始化器对Spring应用上下文进行初始化
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);
        // 向Spring应用上下文中添加异常处理器
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        // 刷新Spring应用上下文
        refreshContext(context);
        afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        // 通知Spring应用的初始化器Spring应用已经启动
        listeners.started(context);
        // 运行Spring应用的初始化器
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        // 通知Spring应用的初始化器Spring应用已经运行
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
1、通过Spring应用上下文初始化器创建Spring应用上下文。
2、通过Spring应用上下文初始化器对Spring应用上下文进行初始化，包括添加异常处理器、设置环境变量等。
3、刷新Spring应用上下文，初始化所有的Bean。
4、运行Spring应用的初始化器，包括CommandLineRunner和ApplicationRunner等。
5、启动完成后，通知Spring应用的初始化器。
6、返回Spring应用上下文。
```


2、spring中bean的生命周期
(a)Bean 实例化：当 Spring 容器接收到 Bean 的定义时，容器会实例化 Bean 对象。
(b)属性赋值：容器会根据 Bean 定义中的属性值或者配置文件中的属性值来为 Bean 对象设置属性。
(c)Aware 接口回调：如果 Bean 实现了 Spring 定义的 Aware 接口，容器会回调 Bean 的相应 Aware 接口方法。
(d)BeanPostProcessor的前置处理：如果容器中存在BeanPostProcessor 的实现类，会在 Bean实例化之后、属性赋值之前调用它们的postProcessBeforeInitialization 方法。
(e)初始化方法调用：如果 Bean 定义中指定了初始化方法，容器会在 Bean 实例化和属性赋值之后调用初始化方法。
(f)BeanPostProcessor 的后置处理：如果容器中存在 BeanPostProcessor 的实现类，会在 Bean 初始化方法调用之后、前置处理之后调用它们的postProcessAfterInitialization 方法。
(g)DisposableBean 的销毁方法调用：如果 Bean 实现了 Spring 定义的 DisposableBean 接口，容器会在关闭时调用它的 destroy 方法。
(h)自定义销毁方法调用：如果Bean定义中指定了自定义的销毁方法，容器会在关闭时调用它。



# 中间件

## 消息队列

### kafka

1、kafka架构

(1) producer: 采用push模式将消息发送到broker
(2) broker: kafka支持水平扩展，一般broker数量越多集群的吞吐量越大
(3) consumer: 采用pull模式订阅并消费消息
(4) zookeeper: 

partition: 物理上的概念，每个topic包含一个或多个partition，一个partition对应一个文件夹，这个文件夹下存储partition的数据和索引文件，每个partition内部是有序的,而不保证topic中不同partition的顺序



ConsumerGroup: 每个consumer属于一个特定的consumer group，可为每个consumer指定group name，若不指定，则属于默认的group，一条消息可以发送到不同的consumer group，但一个consumer group中只能有一个consumer能消费这条消息

kafka是发布与订阅模式，这个订阅者是消费组，而不是消费者实例。每一条消息只会被同一个消费组里的一个消费者实例消费，不同的消费组可以同时消费同一条消息;kafka保证的是稳定状态下每一个消费者实例只会消费一个或多个特定partition数据，而某个partition的数据只会被某一特定的consumer实例消费，这样设计的劣势是无法让同一个消费组里的consumer均匀消费，优势是每个consumer不用跟大量的broker通信，减少通信开销，也降低了分配难度。而且，同一个partition数据是有序的，保证了有序被消费

2、

Kafka和RocketMQ都是基于消息队列的分布式消息中间件，它们的吞吐量取决于多个因素，例如消息大小、网络延迟、磁盘速度、硬件配置等。不过，从一般情况来看，Kafka的吞吐量要高于RocketMQ，主要原因如下：

磁盘顺序写入：Kafka使用磁盘顺序写入的方式来存储消息，而RocketMQ则使用随机写入的方式，因此Kafka的磁盘写入效率更高，能够更快地写入消息，从而提高了吞吐量。

批量发送：Kafka支持批量发送消息，即将多个消息一次性发送到服务器，从而减少了网络IO的次数，提高了吞吐量。而RocketMQ目前还不支持批量发送。

网络IO模型：Kafka使用NIO模型来处理网络IO，而RocketMQ使用的是阻塞IO模型，NIO模型相对于阻塞IO模型来说，具有更高的并发性和吞吐量。

分区和副本机制：Kafka使用分区和副本机制来提高消息的并发性和可靠性，而RocketMQ则使用队列和主从复制机制。分区和副本机制能够更好地支持并发处理和故障容错，从而提高了吞吐量。

需要注意的是，Kafka和RocketMQ的吞吐量并不是绝对的，它们的性能和吞吐量取决于多个因素，例如硬件配置、网络状况、消息大小、消息格式等，因此在实际应用中，需要根据具体情况选择合适的消息中间件，并进行合理的配置和优化，以达到更好的性能和吞吐量。

