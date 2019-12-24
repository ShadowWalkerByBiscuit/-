###  这次的公众号目标是肥朝，主要是针对Redis的一次整理阅读。  

Redis的qps读是11w，写是8w。  

String 最大支持512M。  

zset 每个元素都会关联一个 double 类型的分数。redis 通过分数来为集合中的成员进行从小到大的排序。zset 的元素是唯一的，但是分数（score）却可以重复。  

Redis 事务不保证原子性。简单的说，多条命令中有个别失败的，整体还是会继续执行完成。  

Redis执行事务可以分成三个阶段，1-MULTI开启事物2-命令进入队列3-EXEC执行事务。  

Redis持久化分两种：  
1-RDB（fork一个子线程，同事内存会翻倍）   
2-AOF(Append only file)文件进行写入并不会马上将内容同步到磁盘上，而是先存储到缓冲区，然后由操作系统决定什么时候同步到磁盘。所以这里就会有相应的同步策略（always会影响性能everysec表示每一秒，即使丢失也不会太多no就会丢失数据）。   

主从复制，是通过传输RDB文件给子服务器实现的。使用 slaveof host port 命令来让一个服务器成为另一个服务器的从服务器。   

Redis进行RDB的指令：   
1-Save阻塞进行，期间不处理其他指令，这里可以通过save配置触发的条件（save 900 1 表示900 秒内如果至少有 1 个 key 的值变化，则触发RDB）   
2-bgsave就是异步执行，也是fork一个子线程来完成任务，这里在fork的过程中也是会阻塞的，理论上时间很短，但是由于数据量过大也可能带来长时间的阻塞。   
3-saveCron就是通过定时任务的形式进行备份，具体过程是定时检查save配置的条件是否被满足，如果满足的话就通过bgsave来进行RDB。  

因为AOF是根据时间不断累加的，这样就会造成堆积，Redis提供了精简优化的rewrite方法，生成一份新的AOF文件，保证新旧两份文件对于Redis数据状态来说是一致的，但是新文件通常小得多（只要最后数据一致，多条命令最终状态，那么一条命令就可以替换掉。）AOF 文件重写并不需要对现有的 AOF 文件进行任何读取、分析或者写入操作，而是通过读取服务器当前的数据库状态来实现的。首先从数据库中读取键现在的值，然后用一条命令去记录键值对，代替之前记录这个键值对的多条命令，这就是 AOF 重写功能的实现原理。

Redis常用命令记录学习：   
总结：P开头代表填的时间单位为毫秒，B开头代表是阻塞命令，M代表同时操作多个key，L,R代表对队列的左右操作。pf开头的是操作HyperLoglog。
1-slowlog get x(记录数)获取最近x条慢查询记录。  
2-info memory获取redis内存使用情况，  
used_memory_rss_human代表整体使用的内存，包括内存碎片，redis本身程序使用的内存。  
used_memory_peak_human:43.14M代表最多时候数据使用的总内存。  
maxmemory_human设置的Redis可用的总内存。  
maxmemory_policy内存满的时候使用的淘汰策略。  
mem_fragmentation_ratio=used_memory_rss_human/used_memory_human也就代表了内存碎片的多少，如果太大说明内存碎片太多，如果<1那么说明出现了swap，出现了内存跟硬盘的交换，是比较危险的，因为硬盘的数据跟内存差了很多个数量级会出现阻塞。   

3-expire跟persist相对于，用于设置过期时间跟删除过期时间（永久生效）。  
4-RENAME要注意的是，如果新的名字存在则会被覆盖。RENAMENX就限制了新的名字只能是不存在的，否则报错。   
5-sort默认是对数字进行排序，如果要对字符串排序的话，那么在后面显式的加上ALPHA字段。sort也支持limit offset 和 count 两个参数。  
6-scan返回值为0说明完成了一个完整的遍历，MATCH 可以帮助筛选匹配的值。  
7-get 如果 key 不存在那么返回特殊值 nil 。  
8-MGET就是一次查询多个key。MSET就是一次设置多个值。  
9-SET key value [EX seconds] [PX milliseconds] [NX|XX] ，EX后面跟秒单位，PX后面跟毫秒单位，NX代表key不存在才执行，XX代表key存在才执行。  
10-HKEYS跟HVALS分别获取Hash中的Key跟value值。  
11-LREM vale count 移除count个跟value相等的元素。如果count=0，那么就是移除所有跟value相等的元素。  
12-Redis 的 HyperLogLog 提供了一种不太准确的基数统计方法（比如网站的访问量） ，它的标准误差是 0.81%，只需要 12K 内存。 




