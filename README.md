《后端架构师技术图谱》(Golang)
=======
[![知识共享协议](https://img.shields.io/github/license/simonhgao/back-end-architect)](https://github.com/simonhgao/back-end-architect/blob/main/LICENSE)

* [服务端缓存](https://github.com/simonhgao/back-end-architect/master/README.md#服务端缓存)
	* [Redis](https://github.com/simonhgao/back-end-architect/blob/main/README.md#redis)

### Redis
[《Redis 官方文档》](http://www.redis.cn/)

* [《redis使用指导》](https://www.runoob.com/redis/redis-tutorial.html)
* [《redis客户端详解》](TODO)
* [《redis服务端详解》](TODO)
* [《redis底层原理》](https://blog.csdn.net/wcf373722432/article/details/78678504)
	* 使用 ziplist 存储链表，ziplist是一种压缩链表，它的好处是更能节省内存空间，因为它所存储的内容都是在连续的内存区域当中的。
	* 使用 skiplist(跳跃表)来存储有序集合对象、查找上先从高Level查起、时间复杂度和红黑树相当，实现容易，无锁、并发性好。
* [《Redis持久化方式》](https://github.com/simonhgao/back-end-architect/blob/main/cache/Redis/Redis%E6%8C%81%E4%B9%85%E5%8C%96.md)

	* rdb储存稳定数据，AOF储存未快照数据，也可以两者结合使用。
