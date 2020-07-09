#### 1.jvm：
    http://www.heartthinkdo.com/?p=2987     https://blog.csdn.net/xidiancoder/article/details/70948569
    jmap -heap pid 查看堆信息
    jmap -histo pid 查看对象使用详情
    jmap -dump:format=b, fiele=* pid      堆文件可以用eclipse Memory Analyzer查看
    jps：查看所有java进程
    jstat：查看进程jvm gc情况
    https://my.oschina.net/feichexia/blog/196575
#### 2.sentinel
    热点参数限流：https://github.com/alibaba/Sentinel/wiki/%E7%83%AD%E7%82%B9%E5%8F%82%E6%95%B0%E9%99%90%E6%B5%81
    url清洗https://anjia0532.github.io/2019/03/05/sentinel-restful/