### 继续扫荡公众号的干活，这次的对象叫清幽之地，之前偶尔看过一篇他的文章，觉得不错。   

对于Spring在初始化bean的时候（也就是IOC），主要分两个大阶段，第一个就是bean本身的实例化（从无到有，从字节码的加载，校验，解析，分配内存，设置基本属性比如int类型的初始值为0），第二个阶段就是依赖注入（比如打了注解的属性，或者根据xml文件配置的属性）。  

根据注解，分成注解在属性上，一般都是通过反射对应字段的set方法进行注入。如果注解打在方法放，则是通过调用方法时，将参数进行注入。  

在注入属性的同时还要进行初始化，不然只是一个实例化（没有赋值的空对象），这个时候调用的就是POJO里面的字段里面的set get方法，也可以认为write read方法。所以在创建bean的时候必须加上setter getter。  

对于整个周期，spring保留了扩展的方法，比如常见aware接口（之前一直不知道这是干嘛的，这些方法的参数是谁传进来的，目前理解就是想spring要东西的，比如beanNameAware，beanFactoryAware，beanContextAware，就是告诉在生成这一类型Bean的时候同时提供，让用户进行扩展。）   
还有就是bean生命周期那些PostConstruct,init-method方法，destroyMethod方法，InitializingBean的afterPropertiesSet方法，DisposableBean的destroy方法，BeanPostProcessor的postProcessBeforeInitialization方法跟postProcessAfterInitialization方法。这个有个生命周期的顺序，要记住，面试必考。  

简单罗列：  
第一阶段：加载，校验，解析，初始化，实例化。  
第二阶段：beanNameAware，beanClassLoaderAware,beanFactoryAware。  
第三阶段：BeanPostProcessor.postProcessBeforeInitialization()，@PostConstruct，InitializingBean.afterPropertiesSet(),init-method，BeanPostProcessor.postProcessAfterInitialization()。  
第四阶段：@PreDestroy,Disposale.destroy(),destroy-method,finialize。  

spring有个注明的循环依赖问题（分成构造器，setter两种，构造器的情况是无解的，因为调用构造器的话不能保证单例），简单来说就是A初始化一个对象a的时候，A里面有依赖属性B，那么就要初始化一个b给a，如果B中也有一个属性A，那么就会出现死循环（单例的前提下，如果不是单例，那么完全可以存多个不同的a或者b）。但是这种依赖又是允许存在的，如果是非单例模式，那么这个问题无解，如果是单例模式下解决办法就是提前曝光，设置了3个缓冲区，标注哪些bean是在初始化中，比如上面b要注入a的时候，将a提前曝光，告诉b，直接拿缓存中的a去用就好了不需要帮a初始化了，因为之前a已经在初始化过程中了。   

spring事物中三大类，PlatFormTransactionManager（抽象层，通过数据库配置那里定义注册具体的子类，可以设置事物传播级别），TransactionDefinition（一般使用默认的DefaultTransactionDefinition），TransactionStatus用来表明事物的状态。   


那天晚上阿里的面试，问了一个问题，什么是双亲委派跟如何打破双亲委派。当时只回答出了前半部分，后面部分不知道，今天就看书补充了一下。
双亲委派是在类加载那一层，java中类加载器分成1-启动类加载器(Bootstrap ClassLoader)2-扩展类加载器（Extension ClassLoader）3-应用加载器（Application ClassLoader）4-用户自定义5-特殊的线程上下文类加载器（Thread Context ClassLoader）  
在一个类加载器，接收到加载任务的时候不会马上加载，而是先判断是否加载过了，如果没有就委托给自己的Parent（也就是双亲）去加载，并且依次向上，如果Parent无法加载，才调子类去加载，依次想下。这个就是双亲委派。  
这里要知道每个类加载器是有加载范围的，比如启动类是加载最核心的类有一个专门的路径，同理其他加载器也是，这样就变向形成了加载的优先级，保证最核心的优先加载，这样子可以防止如果用户自定义了一些类跟核心类一样，比如System，String这些，那么就会优先加载java的定义的，并且加载之后用户的就不会再加载或者加载报错了，这样可以防止一样不到的错误。  
并且两个类相等，是要在同一个加载器加载的前提下，比如equals，instanceof这些方法，也都是要在这个前提下。如果是两个不同的加载器加载，就算class文件是一样的，那么最终比较的结果也是不同的。  

打破双亲委派：  
1-classLoader先通过load方法判断是否已经加载过，如果没有那就回网上调用Parent，如果返回错误（也就是无法加载）就会调用findclass方法去加载。所以这里要打破，就可以通过重写loadClass方法来实现。  
2-JNDI（java naming and Dictionary interface）也就是在加载核心类的时候需要加载下层的类的需求（比如说JDBC从原来的手动编写整个过程，到后来的只配置参数就可以直接调用，因为这样多个数据库的驱动是不一致的，系统也不能预先知道），在双亲委派的模式下是无法实现的，做法就是添加ThreadContextClassLoader(默认是Apllication ClassLoader),这样就提供了一个缺口，那么就可以通过SPI继续进行加载。  
3-为了实现热部署。自己手动实现逻辑去加载一个范围里面的类。并不需要遵循双亲委派机制。  

tomcat打破了双亲委派，主要为了解决一下几个问题：  
1. 一个web容器可能需要部署两个应用程序，不同的应用程序可能会依赖同一个第三方类库的不同版本，不能要求同一个类库在同一个服务器只有一份，因此要保证每个应用程序的类库都是独立的，保证相互隔离。
2. 部署在同一个web容器中相同的类库相同的版本可以共享。否则，如果服务器有10个应用程序，那么要有10份相同的类库加载进虚拟机，这是扯淡的。   
3. web容器也有自己依赖的类库，不能于应用程序的类库混淆。基于安全考虑，应该让容器的类库和程序的类库隔离开来。  
4. web容器要支持jsp的修改，我们知道，jsp 文件最终也是要编译成class文件才能在虚拟机中运行，但程序运行后修改jsp已经是司空见惯的事情，否则要你何用？ 所以，web容器需要支持 jsp 修改后不用重启。  

tomcat的做法是，自定义了CommonClassLoader、CatalinaClassLoader、SharedClassLoader和WebappClassLoader则是Tomcat自己定义的类加载器，它们分别加载/common/*、/server/*、/shared/*（在tomcat 6之后已经合并到根目录下的lib目录下）和/WebApp/WEB-INF/*中的Java类库。  

这里就破坏了双亲委派机制，因为自己独立负责加载某一个目录下的全部类而并没有向上委托。  

