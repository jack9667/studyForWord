#### 1.
    假如Redis里面有1亿个key,其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？
    https://maimai.cn/article/detail?fid=1447294778&efid=_nP4tQJ5GDBKngonkAANIg&use_rn=1
    https://zhuanlan.zhihu.com/p/75840305
    https://zhuanlan.zhihu.com/p/32540678
    keys命令可以按指定前缀扫描，但会导致单线程阻塞；可以用scan解决，无阻塞，但是会有一些重复

#### 4.为什么快？
    只有网络请求是单线程的，都是基于内存的
    https://blog.csdn.net/bird73/article/details/79792548
    纯内存访问，作为基础
    单线程操作，避免上下文切换cpu消耗，也没有竞争；这里单线程只在网络请求上使用，像持久化啥的都是多线程（父进程fork一个子进程完成的）
    i/o多路复用，select，poll，epoll三种技术；用的epoll
    select，poll：需要不断的轮寻描述符；而epoll提供了就绪链表，给描述符指定回调函数，会把就绪的设备放入就绪链表里，epoll只要查看链表就行了；
    select，poll每次调用都要做一回用户态到内核态的拷贝；epoll只拷贝一次
    

#### 5.Redis的持久化机制是什么?各自的优缺点?
    https://segmentfault.com/a/1190000002906345

#### 6.数据类型及底层实现
    https://blog.csdn.net/caishenfans/article/details/44784131
    https://i6448038.github.io/2019/12/01/redis-data-struct/
    https://juejin.im/post/5d6bc200f265da03f47c391e

    内部有个redisObject，里面包含类型type，encoding编码方式，指向实际地址的指针
    当内部元素个数少于512，且元素大小小于64字节时 -> ziplist
    zplist：结构：列表总长度+列表末尾元素+元素个数+entry(元素)+尾部界定符；entry内部：上一个元素长度+内容
    string：分int类型和非int类型(字符，浮点，字节)；前者就是int，后者是sds，内部是char[]数组，扩容若小于1M就按需分配，若大于1M，扩充满足后还会多给1M
    list：zplist + 双链表
    hash：zplist+类hashmap，链表发解决冲突，扩容：也有扩容因子+rehash过程
    set：inset + hashmap，没有重复元素，整数类型用inset（有序不重复的数组）
    zset(sortedset)：zplist+跳表(skipList)，zplist内部按照分数来排序(默认升序)；跳表内部有skipListNode结构，包含很多层L和该层的下一个节点，回退指针BW用于回退，score+object本身；score是本身set进来的
    
    
#### 7.redis雪崩，击穿，穿透
    https://blog.csdn.net/zeb_perfect/article/details/54135506
    雪崩：对所有key，key过期时间相同，同一时间全失效，导致请求都到数据库，造成雪崩
    解决：过期时间散列；加锁或者队列
    击穿：对一个key，当它过期时，有大量请求过来，导致击穿
    解决：1.逻辑过期，把过期时间放到value中，用逻辑判断是否过期；2.互斥锁setnx
    穿透：对一个一定不存在的key不停的访问，流量过大数据库崩溃
    解决：布隆过滤器；对不存在的值设置null存入，并设置短的过期时间

##### 7.1集群高可用：
    https://juejin.im/post/5db851706fb9a0203c34de54
    分cluster集群完成分布式，集群机器部多个redis node，以sentinel+mater+slave（哨兵主从）实现高可用，同时读写分离，利用一致性hash 2^32-1长完成数据到master的分布保证负载均衡，node里再进行hash槽分，每个solt保存一部分，2^14个槽是的增加和删除更方便（把key用crc16算法得到结果，和2^14取余进行hash）

    主从同步：mater会将主服务同步给多个从服务，而sentinel维护一个拓扑结构到多个主从节点和其他sentinel节点，做心跳验证；数据分hash槽，每个key值映射到不同的区间内，每个node对应维护一部分区间；
    哨兵：sentinel，作用：集群监控，消息通知，故障转移，注册

    分布式锁：可基于redis和zookpeer；保证多个服务互斥的访问共享的资源；
                   https://juejin.im/post/5cc165816fb9a03202221dd5
                   redis：setnx+expire；先用setnx设置key，只有当key不存在才返回设置成功返回1，否则失败，在设置过期时间保证释放，但这样有问题，在expire出问题将永远部失效；正确做法：set(key, UniqueId, "NX", "EX", seconds)，通过整合setnx 和expire为一个字段，但还有问题，set锁时要设置一个value，保证时间过期后，拿到锁的另一个线程和之前线程获得的不是同一个，要不然前一个线程再苏醒过来会释放掉同一个锁，释放的时候判断锁是不是自己的就行了；

#### 8.redis一致性hash
    将物理机器编号hash到0-2^32长的环上，请求key按顺时针hash落到相应的机器上
    为保证平衡性，添加虚拟节点，就是真实节点的复制replica，均匀的分布，每个虚拟节点对应真实的服务器，避免一台崩掉，发生请求聚集的雪崩

#### 9.为什么 Redis 一定要用跳表来实现有序集合？
    跳表区间查询，首节点命中logN 之后往后面遍历就行；实现更简单，
