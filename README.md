《后端架构师技术图谱》(Golang)
=======
[![知识共享协议](https://img.shields.io/github/license/simonhgao/back-end-architect)](https://github.com/simonhgao/back-end-architect/blob/main/LICENSE)

* [网络](https://github.com/simonhgao/back-end-architect/blob/master/README.md#网络)
 	* [Linux 命令](https://github.com/simonhgao/back-end-architect/blob/main/README.md#Linux 命令)

* [云技术](https://github.com/simonhgao/back-end-architect/blob/master/README.md#云技术)
 	* [Docker](https://github.com/simonhgao/back-end-architect/blob/main/README.md#Docker)

* [缓存](https://github.com/simonhgao/back-end-architect/blob/master/README.md#缓存)
	* [本地缓存](https://github.com/simonhgao/back-end-architect/blob/main/README.md#本地缓存)
	* [Memcached](https://github.com/simonhgao/back-end-architect/blob/main/README.md#Memcached)
	* [Redis](https://github.com/simonhgao/back-end-architect/blob/main/README.md#redis)


## 网络
### Linux 命令
* [《linux常用命令》](https://juejin.cn/post/6844903844267180039)
	* 网络配置： ifconfig、 ip  
	* 连通性探测： ping、 traceroute、 telnet、 mtr
	* 网络连接： netstat、 ss、 nc、 lsof
	* 流量统计： ifstat、 sar、 iftop
	* 交换与路由： arp、 arping、 vconfig、 route
	* 防火墙： iptables、 ipset
	* 域名： host、 nslookup、 dig、 whois
	* 抓包：[tcpdump](https://mozillazg.com/2018/01/tcpdump-common-useful-examples-cookbook.html)
		* 常用命令： sudo tcpdump -iany port 10000 -Xnlps0 
### 构成
* [《NAT》](https://www.jianshu.com/p/62028875d53e)



## 云技术
### Docker
* [《Docker》](https://juejin.cn/post/6844903713115488269)


## 缓存
#### 缓存失效策略
	当缓存需要被清理时（比如空间占用已经接近临界值了），需要使用某种淘汰算法来决定清理掉哪些数据。常用的淘汰算法有下面几种：
	FIFO：First In First Out，先进先出。判断被存储的时间，离目前最远的数据优先被淘汰。
	LRU：Least Recently Used，最近最少使用。判断最近被使用的时间，目前最远的数据优先被淘汰。
	LFU：Least Frequently Used，最不经常使用。在一段时间内，数据被使用次数最少的，优先被淘汰。

### 本地缓存
* [《go-cache》](https://juejin.cn/post/6844903967139299336)
* [《BigCache》](https://pandaychen.github.io/2020/03/03/BIGCACHE-ANALYSIS/)

### 服务端缓存

#### Memcached
* [《Memcached 教程》](http://www.runoob.com/Memcached/Memcached-tutorial.html)
* [《深入理解Memcached原理》](https://blog.csdn.net/chenleixing/article/details/47035453)
	* 采用多路复用技术提高并发性。
	* slab分配算法： memcached给Slab分配内存空间，默认是1MB。分配给Slab之后 把slab的切分成大小相同的chunk，Chunk是用于缓存记录的内存空间，Chunk 的大小默认按照1.25倍的速度递增。好处是不会频繁申请内存，提高IO效率，坏处是会有一定的内存浪费。
* [《Memcached软件工作原理》](https://www.jianshu.com/p/36e5cd400580)
* [《Memcache技术分享：介绍、使用、存储、算法、优化、命中率》](http://zhihuzeye.com/archives/2361)

* [《memcache 中 add 、 set 、replace 的区别》](https://blog.csdn.net/liu251890347/article/details/37690045)
	* 区别在于当key存在还是不存在时，返回值是true和false的。

* [**《memcached全面剖析》**](https://pan.baidu.com/s/1qX00Lti?errno=0&errmsg=Auth%20Login%20Sucess&&bduss=&ssnerror=0&traceid=)

#### Redis
[《Redis 官方文档》](http://www.redis.cn/)

* [《redis使用指导》](https://www.runoob.com/redis/redis-tutorial.html)
* [《redis客户端详解》](TODO)
* [《redis服务端详解》](TODO)
* [《redis内存估算方案》](https://searchdatabase.techtarget.com.cn/7-20218/), [《redis内存预算工具》](http://www.redis.cn/redis_memory/)
	* string类型的内存大小 = 键值个数 * (dictEntry大小 + redisObject大小 + 包含key的sds大小 + 包含value的sds大小) + bucket个数 * 4 （bucket（桶）：table的每一项称为bucket，bucket指向具有相同索引值的键值对链表）
* [《redis底层原理》](https://www.cnblogs.com/kismetv/p/8654978.html)
	* 使用 ziplist 存储链表，ziplist是一种压缩链表，它的好处是更能节省内存空间，因为它所存储的内容都是在连续的内存区域当中的。
	* 使用 skiplist(跳跃表)来存储有序集合对象、查找上先从高Level查起、时间复杂度和红黑树相当，实现容易，无锁、并发性好。
* [《Redis持久化方式》](https://github.com/simonhgao/back-end-architect/blob/main/cache/Redis/Redis%E6%8C%81%E4%B9%85%E5%8C%96.md)
* [《redis缓存失效策略》](https://www.cnblogs.com/dudu2mama/p/11366292.html)

	* rdb储存稳定数据，AOF储存未快照数据，也可以两者结合使用。
