#### 1.幂等性：
    每个producer有唯一pid及一个递增seqNumber，broker端对每个producer维护<pid, topic, partition> 一个递增seqNumber，comit过来的序号比broker维护的大一才接受
    多producer间，用户会提供一个Transaction id同时对应pid，有一个递增num对比新老producer的num值可以抛弃掉老的
#### 2.为什么用消息队列？
    kafka就是一个分布式的消息分发系统；它高吞吐，低延迟，持久性好，高并发，

#### 3.Kafka 的 Topic 和分区内部是如何存储的，有什么特点?
    https://blog.csdn.net/weixin_38750084/article/details/82942564
    消息均匀的分配到每个分区，可以保证负载均衡和水平拓展，topic是逻辑概念，分区是物理概念，每个partition对应一个文件，broker（一台物理机）上可以有多个partition，并且每个分区有leader和flower，leader在写入的同时会复制到flower上，

#### 4.kafka和zookeeper关系？
    broker的id，topic，partition在zookeeper节点中，consumer的offset也在，partition的leader选举机制依赖zookeeper
  
#### 5.选举是如何做的？
    每个broker都有个controller，
    leader在zk上注册临时节点，所有flower监听这个节点，会抢占只有一个成功
#### 6.Kafka 是依靠什么机制保持高可靠，高可用
    高可用：每个partition有learder和flower，分在不同broker上，leader坏了flowser提上来
    高可靠：acks参数，可设置producer那learder和flower都写了才认为写入成功

#### 7.消息队列如何保证消息幂等
    对单个producer，初始化一个唯一pid标示，对该pid下的每一个topic, partition都有一个sequenceNumer对应（从零开始递增），每个broker收到消息时都会赢缓存对比这个number大一才接收，否则抛弃；（保证单producer内部幂等）
    跨session的处理是通过事物保证的，多个topic间传递消息，保证生产和消费数据是一个原子操作；每个应用程序主动设置一个transactionId与pid一一对应，producer通过transactionId拿到pid后有个递增的epoch，旧的会小，可以被抛弃
    
####8.kafka为什么快？
    https://juejin.im/post/5cd2db8951882530b11ee976
    对磁盘的读写是顺序i/o，不会删除数据；内存映射文件mmap，对内存的操作会同步到硬盘上（需要参数设置同步还是异步的flush到硬盘中）；通过sendfile实现零拷贝，内核态不会再切换到用户态，直接通过内存缓冲区到sorket缓冲区进行发送；批量压缩，多个文件压缩；在文件发送时，把所有消息存到多个文件中，一并发送给消费者。

#### 9.怎么保证不被重复消费?
    consumer端：本身自带offest，会标记被消费过的数据，并从上次消费的位置继续消费；这个位置得需要业务方处理入库的重复消费逻辑
    producer端：主要还是通过幂等性，每次被塞到队列里的只有1条

#### 10.怎么保证不丢？
    消费者端：不设置自动提交offeset（原位置01，consumer拿到还没来得及处理down了，自动提交到02位置）
    生产者端：通过acks=all参数，leader只有在写完flower才算提交保证不丢；（写入有同步和异步，异步会写缓冲区然后一并提交 ，并提供回调函数）

#### 11.怎么保证顺序消费?
    发送端重试机制导致的顺序变化，通过幂等操作解决
    消费端同一个队列，或者按keyhash写固定partition
     
#### 12.存储结构？
    partition（每个是有序队列）包含多个segment，segment里面是数据+索引文件
