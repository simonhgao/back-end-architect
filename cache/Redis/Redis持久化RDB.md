《Redis persistence 持久化-RDB》
==============

## 持久化方式
  * RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。
  * AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。
  * Redis 还可以同时使用 AOF 持久化和 RDB 持久化。在这种情况下，当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集，因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。
你甚至可以关闭持久化功能，让数据只在服务器运行时存在。


### RDB
RDB持久化是通过快照的方式，在指定的时间间隔内将内存中的数据集快照写入磁盘。它以一种紧凑压缩的二进制文件的形式出现。可以将快照复制到其他服务器以创建相同数据的服务器副本，或者在重启服务器后恢复数据。RDB是Redis默认的持久化方式，也是早期版本的必须方案。

#### RDB的优点
 * RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。 这种文件非常适合用于进行备份： 比如说，你可以在最近的 24 小时内，每小时备份一次 RDB 文件，并且在每个月的每一天，也备份一个 RDB 文件。 这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。

 * RDB 非常适用于灾难恢复（disaster recovery）：它只有一个文件，并且内容都非常紧凑，可以（在加密后）将它传送到别的数据中心。

 * RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是 fork 出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。

 * RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

#### RDB的缺点
 * 如果你需要尽量避免在服务器故障时丢失数据，那么 RDB 不适合你。 虽然 Redis 允许你设置不同的保存点（save point）来控制保存 RDB 文件的频率， 但是， 因为RDB 文件需要保存整个数据集的状态， 所以它并不是一个轻松的操作。 因此你可能会至少 5 分钟才保存一次 RDB 文件。 在这种情况下， 一旦发生故障停机， 你就可能会丢失好几分钟的数据。

 * 每次保存 RDB 的时候，Redis 都要 fork() 出一个子进程，并由子进程来进行实际的持久化工作。 在数据集比较庞大时， fork() 可能会非常耗时，造成服务器在某某毫秒内停止处理客户端； 如果数据集非常巨大，并且 CPU 时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒。 虽然 AOF 重写也需要进行 fork() ，但无论 AOF 重写的执行间隔有多长，数据的耐久性都不会有任何损失。

#### RDB三种触发方式
* #save命令触发方式（同步）
```
redis> save
OK
```
save执行时，会造成Redis的阻塞。所有数据操作命令都要排队等待它完成。
文件策略：新生成一个新的临时文件，当save执行完后，用新的替换老的。

* #bgsave命令触发方式（异步）
```
redis> bgsave
Background saving started
```
客户端对Redis服务器下达bgsave命令时，Redis会fork出一个子进程进行RDB文件的生成。当RDB生成完毕后，子进程再反馈给主进程。fork子进程时也会阻塞，不过正常情况下fork过程都非常快的。
文件策略：与save命令相同。

save与bgsave对比：
| 命令 | save | bgsave |
| ------------- | ------------- | ------------- |
| IO类型  | 同步  | 异步 |
| 阻塞  | 是  | 是（发生在fork期间） |
| 复杂度  | O(n)  | O(n) |
| 优点  | 不消耗额外内存  | 不阻塞客户端命令 |
| 缺点  | 阻塞客户端命令  | fork消耗额外内存 |

* #规则自动触发方式
某些条件达到时，自动生成RDB文件。
比如我们配置如下：
配置	seconds	changes	说明
```
save	900	1	900秒内改变1条数据，自动生成RDB文件
save	300	10	300秒内改变10条数据，自动生成RDB文件
save	60	10000	60秒内改变1万条数据，自动生成RDB文件
```
以上任一条件达到时，都会触发生成RDB文件。不过这种方式对RDB文件的生成频率不太好控制。如果写量大，RDB生成会很频繁。不是一种好的方式。
修改配置文件：

# 配置自动生成规则。一般不建议配置自动生成RDB文件
save 900 1
save 300 10
save 60 10000
# 指定rdb文件名
dbfilename dump-${port}.rdb
# 指定rdb文件目录
dir /opt/redis/data
# bgsave发生错误，停止写入
stop-writes-on-bgsave-error yes
# rdb文件采用压缩格式
rdbcompression yes
# 对rdb文件进行校验
rdbchecksum yes
不容忽略的触发方式
全量复制
主从复制时，主会自动生成RDB文件。
debug reload
Redis中的debug reload提供debug级别的重启，不清空内存的一种重启，这种方式也会触发RDB文件的生成。
shutdown
会触发RDB文件的生成。


#### RDB快照运作方式
当 Redis 需要保存 dump.rdb 文件时， 服务器执行以下操作：

Redis调用 fork() ，同时拥有父进程和子进程。

子进程将数据集写入到一个临时 RDB 文件中。

当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

如果需要定期保存备份，需要对RDB文件定时上传和备份





Reference：
* [《Redis持久化方式》](http://doc.redisfans.com/topic/persistence.html)
* [《Redis持久化方式与RESP协议》](http://maimai.cn/article/detail?fid=1576337590&efid=OnR8bnJBc1Tj7Sibj6vilw&share_channel=2&use_rn=1)
* [《Redis持久化方式》by 朱哥](https://segmentfault.com/a/1190000012316003)
