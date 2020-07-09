# hbase是一种分布式的，面向列的开源数据库

#### 1.怎么做查询的？
    https://www.jianshu.com/p/52a67e718fe7
    数据是按rowkey字典序排序的，每个region会有startkey和endkey，当一个region过大时会分裂，每个region里面存store，每个store保存一个列簇，在定位到region后进行相应的查找即可（顺序或二分？）
        
#### 2.数据热点和倾斜？
    https://blog.csdn.net/weixin_41279060/article/details/78855679
    数据是按key序排的，当某一类key过多，会导致分裂，总会有region被冷落（倾斜）而对于热点key的region访问会导致请求量过多（数据热点）
    解决：主键做hash当key；翻转时间戳；key要尽可能的小

#### 3.最小存储单位？
    https://cloud.tencent.com/developer/article/1084209
    https://zhuanlan.zhihu.com/p/54184168
    一个hmaster管理多个regionserver（内含多个region），region包含store（每个store保存一个列，一个region包含多个store），store内由hfile保存的storefile（物理存储依赖hdfs）
    hbase不删数据，在原有上已新的时间戳追加
    以cf+colum+vaule完成
