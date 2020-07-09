#自实现LUR结构 
满足put() get() O(1), 且不依赖LinkedHashMap
LUR：最近最少使用缓存策略

缓存淘汰策略，最近最少使用，最近被访问到的最可能被用到，hashmap+双链表，保证get set都是O1，LURcache结构中get，set函数中调用双链表，插入删除和放在最前面
linkedHashMap实现，https://juejin.im/post/5d567642e51d4561f40add67，搞个结构继承它，定义构造函数super(16,0.75f,true); 重写removeEldestEntry判断是否大于容量，super参数16 初始大小，0.75f负载因子会计算出一个threshold控制扩容，true为指定遍历顺序 为顺序，https://www.cnblogs.com/a294098789/p/5323032.html

```java

```