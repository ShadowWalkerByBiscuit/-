# Redis深入笔记

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

String 
最基础的类型， 可以保存图片，二进制对象等信息，最大512m。

hash
hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

list
Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

set
Set是string类型的无序集合。集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

zset(sorted set：有序集合)
zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
zset的成员是唯一的,但分数(score)却可以重复。  

主从复制，在slave节点上配置slaveof host port，可以执行命令，也可以通过配置。  
使用 info replication 命令查看相关的复制状态。  
节点默认都是只读模式，即 slave-read-only 的值默认为 yes。  
如果想要断开复制关系，可以通过在从节点上执行 slaveof no one。  
可以随时通过slaveof命令。切换主节点，但是只要一切换，那原来数据就会被清空之后才开始新的数据复制。  

