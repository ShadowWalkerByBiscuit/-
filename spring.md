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

spring有个注明的循环依赖问题，简单来说就是A初始化一个对象a的时候，A里面有依赖属性B，那么就要初始化一个b给a，如果B中也有一个属性A，那么就会出现死循环（单例的前提下，如果不是单例，那么完全可以存多个不同的a或者b）。但是这种依赖又是允许存在的，解决办法就是提前曝光，设置了几个缓冲区，标注哪些bean是在初始化中，比如上面b要注入a的时候，将a提前曝光，告诉b，直接拿缓存中的a去用就好了不需要帮a初始化了，因为之前a已经在初始化过程中了。   

spring事物中三大类，PlatFormTransactionManager（抽象层，通过数据库配置那里定义注册具体的子类，可以设置事物传播级别），TransactionDefinition（一般使用默认的DefaultTransactionDefinition），TransactionStatus用来表明事物的状态。   





