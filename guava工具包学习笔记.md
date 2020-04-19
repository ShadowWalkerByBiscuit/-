# guava读书笔记  

通过集合类创建集合如：  
// 普通Collection的创建
List<String> list = Lists.newArrayList();
Set<String> set = Sets.newHashSet();
Map<String, String> map = Maps.newHashMap();  

通过不可变集合创建集合如（线程安全，不可变）：  
ImmutableList<String> iList = ImmutableList.of("a", "b", "c");
ImmutableSet<String> iSet = ImmutableSet.of("e1", "e2");
ImmutableMap<String, String> iMap = ImmutableMap.of("k1", "v1", "k2", "v2");  

map中包含list：  
Multimap<String,Integer> map = ArrayListMultimap.create();		
map.put("aa", 1);
map.put("aa", 2);
System.out.println(map.get("aa"));  //[1, 2]

双键值map：  
Table<String, String, Integer> tables = HashBasedTable.create();  

集合操作类：
Joiner 跟Splitter

判断操作类：  
Preconditions  

字符串处理器：  
CharMatcher

优雅的自定义toString：  
MoreObjects

排序器：  
Ordering  

程序计时器：  
Stopwatch   

文件操作：  
Files 

这玩意不用去记，主要是知道有这么个东西，然后遇到问题的时候往这个方向去想，再去具体的工具类找自己有用的。
