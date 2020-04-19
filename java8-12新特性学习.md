
# java8特性  
1 接口方法的默认实现
通过关键字default来定义。  
会出现一个问题，如果一个类实现多接口，碰巧多个接口中有同名的默认实现方法会出现冲突，解决办法就必须重写方法明确具体是调用那个方法。

2 接口编写静态方法  
跟类方法一样。

3 lambda表达式跟函数式编程
@FunctionInterface 类中只有一个未实现接口，但是允许存在默认实现方法，方法调用（::）  

4 Stream流
分为中间操作跟终结操作，要注意stream的整体的执行顺序  

5 Optional判断结果（要用起来）

6 Date/time API的改进
LocalDate LocalDateTime LocalTime DateTimeFormatter

7 String类中提供了join方法来完成字符串的拼接

# java9特性 
1 集合工厂方法
用于创造一些不能修改的集合  

2 接口中的私有方法

# java10特性
1 var 局部变量类型推断（像js一样）
但是对于代码阅读是很不友好的事情。

# java11LTS（Long Term Support）  
1 本地变量类型推断

2 字符串加强（提供方法做处理，如去掉空格）

3 stream 新方法
takeWhile dropWhile（就是一个判断）  
.takeWhile(n -> n < 3)  表示n<3成立都运行  
.dropWhile(n -> n < 3)  表示n<3不成立才开始运行  

总结：
对于程序员来说，要注意跟学习的新技术不多，每个版本都会有gc的优化，性能指标也是影响java发展的最重要因素了。

