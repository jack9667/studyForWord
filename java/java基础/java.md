#1.Java部分：Java基础，集合，并发，多线程，JVM，设计模式

##2.hashmap和hashtable的区别？
    hash需要保证key是不可变对象
    https://blog.csdn.net/u010983881/article/details/49762595
    继承不同
    hashtable线程安全，map不安全，hashtable内部对put，get，remove做了synchronized，
    hashtable不允许key value有null，map可以
    hashtable直接使用hashcode，map做了二次hash
    初始化值不同
    arraylist线程不安全，vertor是安全的，同样是做了synchronized
    Collections的synchronizedMap方法使HashMap
###2.1 LinkedHashmap
    保证插入顺序，遍历是顺序访问的，通过参数可以实现LRU
###2.2 TreeMap
    按key排序，默认自然序(升序)，底层红黑树，输出的时候有个successor利用中序便利保证顺序
###2.3 Collections
    collections是工具类，提供了一系列静态接口
    collection是基本的集合接口

###2.4 HashMap
     初始值16
     hash位置，>>：右移除以2 <<：左移乘以2  >>>：无符号右移，不管符号位都补0     
     先计算key的hashcode，再对hashcode进行桶长度取模，jdk1.7：h&(length - 1) jdk1.8：h^(h>>>16) 高16位亦或低16位，hashcode()，
     resize：扩容是两倍，扩充两倍，将原来的值重新hash到新数组，jdk1.7是头插法重新hash，jdk1.8因为长度是按2次幂扩增的，实际只看变化为是0还是1,0不动，1就变化为原位置+扩容值，多线程没办法保证容易形成环形链表
    负载因子的作用：平衡时间和空间损耗，其过小hash冲突小时间得到保证但空间损耗；反之空间

##2.5 ConcurrentHashMap
    https://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/index.html
    其实现高并发锁，相比hashtable和Collections.synchronizedMap都是通过一个全局锁来阻塞所有线程；
    1.7 而CHM是通过分离锁的方式实现，内部维护一个16长的semgment数组，每个segment包含若干个散列的桶，且链表的next指针为final，只能再头结点插入，value也是volite的对所有内存可见
    1.8 的是对node节点和tree节点做cas和synchronized保证并发安全
    CHM内部没有null值，key和value，读到为空的value值说明发生重排序，会阻塞重读
    https://www.cnblogs.com/loren-Yang/p/7466111.html
    size操作：baseCount作为个数，put和remove都会调用addcount函数，当cas操作失败时，会把x的值放入counterCell[]中，size()函数会遍历所有countCell[]数组的值+baseCount作为最终的值

##3.JVM的内存模型，回收机制    
    堆、栈、本地方法栈、方法区、寄存器
    堆：所有对象实例
    栈：基本数据类型、对象的引用、方法的入口指针等
    本地方法栈：和栈一样，只不过是对本地运行的方法的
    方法区：静态变量、常亮、已被加载的类信息等
    cms+g1：垃圾收集器，https://juejin.im/post/5dad5621f265da5bab5bda33
     https://www.jianshu.com/p/bdd6f03923d1
##4.wait和sleep的区别
     https://www.jianshu.com/p/25e959037eed

    wait在synchronize代码块中调用针对加锁的对象需要notify notifyall唤醒h会释放montior或者锁sleep针对线程不释放锁


    yield暂停当前线程，让在等待的优先级更高的线程来执行，如果没有就继续执行


##5.synchronized
    底层原理 是可重入锁吗？CAS原理
    synchronized关键字，通过锁的机制来实现的同步
    https://blog.csdn.net/javazejian/article/details/72828483
    修饰代码块、方法、静态方法、类
    修饰static方法相当于锁住了这个类，该类所有对象拥有同一把锁；普通方法是锁住了调用这个方法的对象
    不能继承，每个对象拥有一个monitor，通过对它的获取互斥实现锁
    是可重入锁， 同一个线程重复请求由自己持有的锁对象时，可以请求成功而不会发生死锁。
    通过计数器，获取时是同一个线程id，就+1，释放-1，避免死锁

    乐观锁：假设访问资源不会发生冲突，就不会阻塞，在更新数据时会判断数据是否被更新过；cas是其一种实现，
    CAS(V，O，N) 如果ov相同，没被改，把n写入v；如果ov不同，直接返回v，失败会一直重试；v内存的值，o预期值，n要写入的值
    ABA问题，加版本号区分，都相同才相同
    https://www.jianshu.com/p/d53bf830fa09
    悲观锁：每个资源只给一个线程使用，其他会阻塞，直到被释放，synchronized悲观锁
    https://blog.csdn.net/qq_39839075/article/details/100055872

##5.1volatile
     https://blog.csdn.net/suifeng3051/article/details/52611233
     相比于synchronized区别，synchronized是执行控制，通过获取对象的monitor来实现锁的机制，其他线程无法获取，同时有内存屏障，保证被修饰的代码块都会刷入内存（实现happens-before）
     volatile主要是解决变量的内存可见性，保证变量的修改对所有内存可见，它禁止指令重排序，但是它无法保证操作的原子性
     happens-before：是一种偏序关系，线程A的操作先于B执行
    
##6.锁的类别、锁的状态
    https://www.cnblogs.com/aspirant/p/11470858.html
    乐观锁，悲观锁，轻量级锁，偏向锁，重量级锁，自旋锁
    轻量级锁：引入轻量级锁的主要目的是 在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。
    自旋锁：一个线程尝试获取锁时，如果被占用会一致循环检查，而不是进行线程的挂起，已减少cpu处理
##7.举个例子HashMap怎么线程不安全？和treemap区别？
   hashMap多线程写不安全
   treemap：按key排序，红黑树
   linkedhashmap：插入有序，hashmap+双链表，实现lru
##8.volatile关键字？
    https://www.cnblogs.com/dolphin0520/p/3920373.html
    题：https://juejin.im/post/5dc4d7c75188257bd71318d8
    修饰变量，修改会强制写入内存，保证对其修改所有线程可见，但不保证原子性
    其禁止指令重排序，通过保证指令执行顺序完成的

##9.红黑树和平衡二叉树AVL
    插入删除查找都是 log n
    红黑树：可以理解为平衡二叉树的子集，不保证高度平衡，
                 只有红黑节点；根节点黑；叶节点黑 ；任意路径上黑节点数目相同；红节点孩子必须是黑的；
    平衡二叉树：左右子树高度差不超过1，频繁的插入和删除会导致性能下降，维护平衡代价高

##10.java主流锁
    synchronized是通过锁的方式实现同步，
    lock接口有很多实现类， ReentrantLock ， ReentrantReadWriteLock.ReadLock ， ReentrantReadWriteLock.WriteLock
    https://tech.meituan.com/2018/11/15/java-lock.html
     
##锁的状态：
    对synchronized锁的状态：对象头+monitor
    无锁：不锁住资源，多个线程只能有一个修改资源，失败会重试
    偏向锁：一个同步代码块一直被一个线程访问，改线程会自动获得改锁；-XX:-UseBiasedLocking=false，关闭之后程序默认会进入轻量级锁状态。通过对比Mark Word解决加锁问题，无cas操作
    轻量级锁：多个线程争抢资源时，没抢到就自旋等待锁释放，cas+自旋
    重量级锁：没抢到就阻塞。其他线程无法获取资源

##锁的膨胀：
    只有一个线程，偏向锁，通过mark word id判断是不是该线程，是 就会直接获取
    当另一个线程进来，通过cas操作修改mark word id来获取锁，此时临界区多个线程交替，升级为轻量级锁
    当多线程同时进入临界区，轻量级锁通过多次cas自旋无法获取资源就膨胀为重量级锁

##锁的种类：
    乐观锁：线程不锁资源，cas
    悲观锁：线程锁住资源，synchronized
    自旋锁：锁住同步资源失败，线程不阻塞，-XX:PreBlockSpin设置自旋次数，要不太多浪费资源

    可重入锁：同一个线程在外层方法获取锁后，进入内层方法依然会获得改锁，避免死锁，reentrantlock，synchronized：有一个计数器，为0时其他线程才能拿，获取改锁的线程在进入递归后对该计数器+1，退出-1
    公平锁：多个线程强资源，排队
    非公平锁：先尝试插队，插队失败再排队
    共享锁：多个线程共享一把锁

##11.threadLocal
    https://juejin.im/post/5ac2eb52518825555e5e06ee
    实现了线程的数据隔离，修饰的变量只能被当前线程访问，其他线程无法进行访问和修改
    通过set，get
    每个thread(线程)都有一个threadLocalMap对象，key是threadLocal这个变量对象，value是对应的值，threadLocal本身不存储，
    set方法会找到当前线程的threadLocalMap并set进value值，如果没有就new一个
    get方法同样，根据key返回值
    threadLocal可以理解为创建了一个变量，这个变量被对象+值存到线程的threadLocalMap内，以此来隔绝其他线程

##12.nio
    同步非阻塞io，通俗的理解：https://blog.csdn.net/L_BestCoder/article/details/79352022
    io就是input/output，接发数据，nio -> new i/o
    传统同步阻塞io，是一个链接一个线程，客户端有链接请求就建立一个线程，在数据未就绪之前，一直阻塞(释放cpu)，直到数据就绪，把数据从内核传递给用户端，继续下面的代码(同步）
    nio：是多个链接一个线程，客户端的请求会发到多路复用器上，它轮寻到有io请求才启动线程处理；客户端的io请求会直接返回结果(失败信息)，而同步是客户端线程主动轮寻到发现数据就绪
    channel注册到selector上，通过轮寻selector()遍历事件，处理并回调；channel是基本对象，代表和实体的链接

##13.Lock、ReentrantLock、ReentrantReadWriteLock
    Lock：是基于语言层面的锁，可以被中断，而synchronized是基于jvm的
    ReentrantLock：基于代码层面的锁，需要手动释放；可实现公平和非公平锁；都是可重入的阻塞同步锁；
    ReentrantReadWriteLock：读写锁，读写分离；锁可降级：先拿到写锁，然后获取读锁，释放写锁后就变成读锁了；

##15.hash冲突方法
    再hash
    链地址
    开放地址：线性或非线性，线性就是直接在下一个位置判断，非线性制定一些规则
    建立公共溢出区：所有溢出的都在区里

##16.cms g1
    https://www.cnblogs.com/aspirant/p/8663897.html
    主要参数：
    cms：初始堆大小、最大堆的大小、新生代大小（分eden，survivor）、持久代初始和最大大小、老年代fullgc的比例
    基于 标记-清除 ，造成空间碎片
    stoptheword：停下除gc之外的所有线程、初始标记和重新标记都会stw，但是时间很短
    初始标记：stw
    并发标记：
    重新标记：stw
    并发清理：
    CMSInitiatingOccupancyFraction：指定老年代fullgc的触发阈值

    g1：
        虽然还会保留，年轻代+老年代概念，但不再三大块分内存了，而是被分为很多小块，
        初始标记：stw
        并发标记：
        最终标记：stw
        筛选回收：stw     
        用户可以指定期望停顿时间

##17.内存溢出和内存泄露：
    https://blog.csdn.net/jingzi123456789/article/details/84196357
    内存溢出，oom：当申请内存时，计算机没有空间可以提供
    内存泄露：new了对象，但对象本该被回收时，仍有其他对象在引用导致停留在堆内


##18.threadPool
    https://juejin.im/post/5d1882b1f265da1ba84aa676
    ThreadPoolExecutor
    参数：核心数，最大数，非核心线程存活时间，阻塞队列，线程factory，handler拒绝策略
    逻辑：核心线程会一直存在，提交任务若大于核心数放阻塞队列，若阻塞队列满了，新创建非核心线程处理，若大于最大数拒绝
    拒绝策略：抛异常、直接拒绝、抛弃掉最老的执行当前的、
    工作队列：数组、linked、delay延迟队列、优先级队列；newFixedThreadPool使用了无界的阻塞队列LinkedBlockingQueue不设置大小会一直创建导致内存溢出

## 19.设计模式
    单例模式：
    工厂模式：
    装饰器模式：
    观察者模式：
    中介者模式：

## 20.并发编程：
    原子性：
    可见性：
    一致性：
