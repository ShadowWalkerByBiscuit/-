打算看看dubbo的源码，做个简单的读书笔记。

#Dubbo官方手册阅读

架构：
组件：1.container 2.provider 3.register 4.consumer 5.monitor  
值得注意的是，consumer与register之间采用了异步订阅模式，当服务上线或者下线，获取对应的地址。  
Dubbo目前支持4种注册中心,（multicast,zookeeper,redis,simple，nacos） 推荐使用Zookeeper注册中心  
[register简介](https://www.cnblogs.com/duanxz/p/3772765.html)    
simple作为注册中心的意思，起一个服务作为注册中心，这样的好处是减少第三方的依赖。  

从手册中摘要的主要流程：
调用关系说明
1.服务容器负责启动，加载，运行服务提供者。  
2.服务提供者在启动时，向注册中心注册自己提供的服务。  
3.服务消费者在启动时，向注册中心订阅自己所需的服务。  
4.注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。  
5.服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。  
6.服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。  

值得注意的点：  
1.服务之间最后的调用，register在提供相应地址后不参与调用过程。  
2.monitor上报的数据先缓存在内存，隔一段时间在汇总上报。  
3.注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外。  
4.注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表。同时缓存也会出现数据不一致问题。  

配置文件加载的优先级：  
-D > .xml > .property(-D的优先级最高)  

dubbo提供多种配置方法，个人觉得最优雅的还是注解的形式。
其中关键主键如下：  
@Service（这个包名不是spring的，是阿里的，跟spring一样还要配置扫描的路径，为的是暴露服务跟引用服务）
@Reference

除了注册中心之外，还有一个配置中心，在服务治理（如lb），基础服务配置（就是property写的配置，如dubbo.properties中内容）提供配置。  
zk也可以承担这个角色（config分支存储）。  
值得注意的是，多个配置同时存在（如本地，外部），外部默认有更高的优先级，即会覆盖本地的配置。(后面提到，最高优先级是jvm -D参数的配置)  
（-Ddubbo.config-center.highest-priority=false）取消优先  

配置的编写也分级别：  
1.应用级别（对整个应用其作用）
2.服务级别（对应的服务起作用）
3.多配置级别（有多个对接的应用，如多个注册中心，多个配置中心）
摘抄的具体公式如下
## 应用级别
dubbo.{config-type}[.{config-id}].{config-item}={config-item-value}
## 服务级别
dubbo.service.{interface-name}[.{method-name}].{config-item}={config-item-value}
dubbo.reference.{interface-name}[.{method-name}].{config-item}={config-item-value}
## 多配置项
dubbo.{config-type}s.{config-id}.{config-item}={config-item-value}

## check="true"
用于启动时，检查服务是否可用（服务有可能已经是不可用的，但是本地缓存着之前的地址）

dubbo.reference.check=false，强制改变所有 reference 的 check 值，就算配置中有声明，也会被覆盖。
dubbo.consumer.check=false，是设置 check 的缺省值，如果配置中有显式的声明，如：<dubbo:reference check="true"/>，不会受影响。
dubbo.registry.check=false，前面两个都是指订阅成功，但提供者列表是否为空是否报错，如果注册订阅失败时，也允许启动，需使用此选项，将在后台定时重试。

## 集群容错
cluster进行设置（集群节点级别统一配），value如下  
1.Failover 失败重试，可以配置重试次数（retries）  
2.Failfast 快速失败，失败直接报错  
3.Failsafe 失败忽略  
4.Failback 后台记录失败请求，定时重发  
5.Forking 并行调用，同时请求多个，（通常是读操作，fork可以设置并行数）  
6.Broadcast 广播通知，逐一调用，任一一台报错就报错。  

## 负载均衡
loadbalance进行设置（方法级别），value如下：  
1.Random 随机。  
2.RoundRobin  轮询。  
3.LeastActive  最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。  
4.ConsistentHash 一致性 Hash，相同参数的请求总是发到同一提供者。  

## 线程模型  
在从接口到实现类的那一层，有一个分发器（dispatcher），有点像网络连接走mvc，分发器通不过不同的分发策略来决定是否加入线程池。

##  服务分组  
允许一个接口有多个实现，因为并不关心参数，所以无法通过签名区分，那么就通过制定分组（类似于制定命名空间）的方式解决问题。  

## 几个比较有意思的配置：  
1.结果缓存 cache  
2.回声测试 直接强转EchoService，然后调用echo方法。  
3.隐式参数 RpcContext.getContext().setAttachment("index", "1"); // 隐式传参，后面的远程调用都会隐式将这些参数发送到服务器端，类似cookie，用于框架集成，不建议常规业务使用  RpcContext.getContext().getAttachment("index");   

##  泛化接口调用  
generic参数配置，其实就是通过反射的机制来参数具体的方法名，参数类型，参数值，调用。通过ReferenceConfig创建一个容器，设置要调用的接口路径字符串，参数，并设置generi=true就可以了。返回接口也均封装成map的形式。我个人认为是为了改变方法的签名跟方法名的一种方式。

##  上下文信息  
RpcContext 

## 异步调用
首先要求服务提供方，定义接口返回值为CompletableFuture<>。  
服务使用方，需要增加回调，
```java
// 调用直接返回CompletableFuture
CompletableFuture<String> future = asyncService.sayHello("async call request");
// 增加回调
future.whenComplete((v, t) -> {
    if (t != null) {
        t.printStackTrace();
    } else {
        System.out.println("Response: " + v);
    }
});
// 早于结果输出
System.out.println("Executed before response return.");

//使用RpcContext的方式 此调用会立即返回null
asyncService.sayHello("world");
// 拿到调用的Future引用，当结果返回后，会被通知和设置到此Future
CompletableFuture<String> helloFuture = RpcContext.getContext().getCompletableFuture();
//或者换一种方式调用
CompletableFuture<String> future = RpcContext.getContext().asyncCall(
    () -> {
        asyncService.sayHello("oneway call request1");
    }
);

future.get(); 
```

sent="true" 等待消息发出，消息发送失败将抛出异常。
sent="false" 不等待消息发出，将消息放入 IO 队列，即刻返回。  
return="false" 不关心返回值，直接忽略返回值。  

 AsyncContext 也可以用来提供异步访问   

## 参数回调
在注解里面配置callbacks methods，添加服务完成后的回调操作。  

##  事件驱动  
在接口调用的不同阶段分别会触发oninvoke、onreturn、onthrow三个时间，意味着我们可以指定这个三个事件发生时的动作。

## 本地存根
stub 在服务提供方进行一次包装，做容错，其实就在做了一层代理，在发起访问前做一些通用的校验一些的事情。

## 本地伪装
mock 用于服务降级，也是一层代理，在服务提供方挂掉的时候体统一些通用的返回，比如说404，或者是exception，这时候就可以做统一处理。

##  延迟暴露  
delay=-1 表示等待spring初始化结束之后才向外暴露，设置其他数据为启动延迟的毫秒数，最新版本是默认在spring启动完成后才暴露。
在 Spring 解析到 <dubbo:service /> 时，就已经向外暴露了服务，而 Spring 还在接着初始化其它 Bean。

##  粘滞连接
目的是请求总是打到同一台服务器，相对的是随机或者是不同的服务提供方。sticky="true"  

##  服务降级
对于微服务的治理，分熔断隔离，降级，限流。核心思想是，节省不必要的资源消耗，保证核心功能的正常使用。  

# 优雅停机
前提是使用kill pid，如果有-9 就不行。
使用的是JDK 的 ShutdownHook （通过Runtime类的addShutdownHook方法）;  

## 服务治理  
通过telnet访问dubbo服务，可以通过命令行进行操作。

## 服务容器  
单独以standalon的形式，加载程序，并对外暴露服务。相对于用tomcat启动，可以减少不必要的资源开销。  
默认只加载spring，根据dubbo的spi技术，找到对应文件里面的配置，加载对应容器。  

## Java序列化  
使用Kryo和FST  

## 服务暴露  
核心两个类的核心方法为ServiceConfig.export()，ReferenceConfig.get()    
get的底层是RegistryProtocol.refer()从配置中心获取url,在根据dubbo:// 协议头识别，调用DubboProtocol.refer()；

## 可扩展点
dubbo不是基于java原生的spi技术，只是核心思想是一致的，做到接口的动态扩展，典型的例子就是lb的配置实现。
相较与java的ServerLoader，dubbo对应的扫描加载工具叫 ExtentionLoader。
默认的扫描路径为：  
META-INF/dubbo/internal  
META-INF/dubbo  
META-INF/services  
java写文件的规则是在MEAT-INF/Service下创建一个文件，文件名为接口的完整名（包括全部的路径名，如org.apache.spi.Robot）
内容为实现了该接口的实体类的全路径（可以存在多个实现类）
dubbo的不同点在于文件内容是k-v形式，以提供根据参数化来实现动态配置。如 bumblebee = org.apache.spi.Bumblebee

其中主要的注解为@SPI(RandomLoadBalance.NAME)，@Adaptive("loadbalance")
@SPI的值为默认实现，@Adaptive的值为url中传递的k-v参数中的key，对应的value就是代替@SPI中的默认值，也就是上面提到文件中的key，决定了具体加载的实现类。@Adaptive中的值可以是多个，一次查找，找不到第一个就接着找第二个。  

## 扩展点的实现
通过ExtensionLoader.getExtensionLoader(xxx)获取实例，然后根据xxx去找寻要加载的实体类，首先查找缓存中是否已经加载过，如果没有在缓存中，则创建。
创建的过程是通过扫描文件中对应xxx的实现类的全路径，运用反射机制创建实例，创建完成之后还要注入依赖。接下来要包装成wrapper类。

## 查询实体类过程
首先解析了@SPI注解，把默认实现的value值缓存起来，然后依次去扫描默认的文件夹，然后解析里面的k-v对，然后将v指定的实现类加载并加入缓存。

## @Adaptive 
可以出现在类或者方法上。主要还是在方法上，如果某个某个实现类被 Adaptive 注解修饰了，那么该类就会被赋值给 cachedAdaptiveClass 变量。这样就会在加载的时候，访问缓存不为空，直接返回。这种情况是@SPI，跟@Adaptive都没有值的时候的时候，由实现类提供默认实现。Dubbo 要求该接口至少有一个方法被 Adaptive 注解修饰。这样才存在扩展加载的意义。Dubbo 不会为没有标注 Adaptive 注解的方法生成代理逻辑，对于该种类型的方法，仅会生成一句抛出异常的代码。
@Adaptive({client, transporter})值可以是一个数组，第一个没有找到，则第二个起作用。

## 服务导出  
Dubbo 服务导出过程始于 Spring 容器发布刷新事件，Dubbo 在接收到事件后，会立即执行服务导出逻辑。(容器启动发布事件，通过事件监听触发)  
scope = none，不导出服务
scope != remote，导出到本地
scope != local，导出到远程
最重要的是Invoker 的创建，核心就是对外准备好。
导出本地跟导出远程的区别在于，远程需要向注册中心注册（注册的前提是已经创建了注册中心。），注册的过程也就是想类似与zk的注册中心，写入url等关键信息，让外界能够找到。
在注册的同时还会维护一些逻辑，比如监听器，订阅，心跳等。同时实例化对外的transportor，默认是netty，通过netty的创建，也就打通了对外的通道。

## 服务导入
服务导入，其实就是在服务调用方，使用方法是，如何注入，有初始化的时候就已经加载好相关配置，另外一种就是使用的时候加载，也就是俗称的饿汉跟懒汉。默认是懒汉，可以通过配置成饿汉。
导入有三种1:本地2.直连3.注册中心，但是最后得到的都是一个Invoker。
Invoker 是 Dubbo 的核心模型，代表一个可执行体。在服务提供方，Invoker 用于调用服务提供类。在服务消费方，Invoker 用于执行远程调用。
这章也很难，理解了个大概。
[书签](http://dubbo.apache.org/zh-cn/docs/dev/principals/code-detail.html)

##  魔鬼在细节  
1 防止空指针跟越界异常  
2 时刻考虑多线程的情况会不会出现异常，因为小范围的测试是没办法暴露的。  
3 尽早失败和前置断言，就是强制要求传参的时候提前做校验。  

### dispatch分配策略
all 所有消息都派发到线程池，包括请求，响应，连接事件，断开事件，心跳等。  
direct 所有消息都不派发到线程池，全部在 IO 线程上直接执行。  
message 只有请求响应消息派发到线程池，其它连接断开事件，心跳等消息，直接在 IO 线程上执行。  
execution 只请求消息派发到线程池，不含响应，响应和其它连接断开事件，心跳等消息，直接在 IO 线程上执行。  
connection 在 IO 线程上，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池。  

### ThreadPool
fixed 固定大小线程池，启动时建立线程，不关闭，一直持有。(缺省，注意的是这个跟jdk的fixThreadpool是不一样的。)  
cached 缓存线程池，空闲一分钟自动删除，需要时重建。  
limited 可伸缩线程池，但池中的线程数只会增长不会收缩。只增长不收缩的目的是为了避免收缩时突然来了大流量引起的性能问题。  
eager 优先创建Worker线程池。在任务数量大于corePoolSize但是小于maximumPoolSize时，优先创建Worker来处理任务。当任务数量大于maximumPoolSize时，将任务放入阻塞队列中。阻塞队列充满时抛出RejectedExecutionException。(相比于cached:cached在任务数量超过maximumPoolSize时直接抛出异常而不是将任务放入阻塞队列)    

服务暴露有三种形式：不暴露，本地，远程。dubbo默认是本地加远程。本地暴露的话，就是遵循JVMProtocol，也就相当于注册中心在本地。    
在服务暴露的时候，分两步连接，一步是通过构建netty来监听服务端口，另外一步是通过curator或者zkclient 连接zk来把服务信息写入。  

在服务引用的时候，因为继承了FactoryBean，那么当注入的时候，就会通过工厂方法来获取，这时候就会通过代理，来决定服务是如何注入的，这里又会涉及到是懒汉式还是饿汉式，通过配置参数init来决定，默认是懒汉式也就是一开始就加载了。   
在这个过程中，会缓存服务的信息，构建服务目录。   

dubbo的服务异步同步调用，区别在于，如果是同步，那么调用future的get方法阻塞等待，直到拿到结果为止，而异步调用，直接将结果注册到RPCcontext,做标记，让后面有需要的时候根据key去拿，但是不一定保证每次拿都拿得到。



