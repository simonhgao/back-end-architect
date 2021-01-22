《Redis persistence 持久化》
==============
## 持久化方式
  * RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。
  * AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。
  * Redis 还可以同时使用 AOF 持久化和 RDB 持久化。在这种情况下，当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集，因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。
你甚至可以关闭持久化功能，让数据只在服务器运行时存在。


## RDB
* [RDB详解](https://github.com/simonhgao/back-end-architect/blob/main/cache/Redis/Redis%20Database%20File(RDB).md)
          *RDB方式：定期备份快照，常用于灾难恢复。优点：通过fork出的进程进行备份，不影响主进程、RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。缺点：会丢数据。






Reference：
* [《Redis持久化方式》](http://doc.redisfans.com/topic/persistence.html)
* [《Redis持久化方式与RESP协议》](http://maimai.cn/article/detail?fid=1576337590&efid=OnR8bnJBc1Tj7Sibj6vilw&share_channel=2&use_rn=1)
* [《Redis持久化方式》by 朱哥](https://segmentfault.com/a/1190000012316003)
