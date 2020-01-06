###  继续扫荡公众号的干活，这次的对象叫清幽之地，之前偶尔看过一篇他的文章，觉得不错   

首先确定一下，Mybatis的作用是在Jdbc的基础上进行的了封装，也就底层还依旧是Jdbc。从我们写的Dao层的接口跟Mapper的关系对应，很明确的知道这里肯定是存在代理的。  

Mybatis设置了两个级别的缓存，一级缓存是session级别，二级缓存是mapper级别（意味着可以跨缓存）。二级缓存要打开，一级缓存是默认开启的。  
SqlSource操作原生sql的api对象，生成的最终sql与相关参数用BoundSql来存储。  

SqlSessionFactory用于创建sqlsession，Configuration用于管理配置属性，包括解析后的mapper对象（MappedStatement），接口与mapper的映射关系。
