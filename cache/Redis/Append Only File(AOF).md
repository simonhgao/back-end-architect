
《AOF 日志备份》
==============

AOF就是写日志，每次执行Redis写命令，让命令同时记录日志（以AOF日志格式）。Redis宕机时，只要进行日志回放就可以恢复数据。

 
### AOF的优点
* 使用 AOF 持久化会让 Redis 变得非常耐久（much more durable）：你可以设置不同的 fsync 策略，比如无 fsync ，每秒钟一次 fsync ，或者每次执行写入命令时 fsync 。 AOF 的默认策略为每秒钟 fsync 一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（ fsync 会在后台线程执行，所以主线程可以继续努力地处理命令请求）。
* AOF 文件是一个只进行追加操作的日志文件（append only log，因此对 AOF 文件的写入不需要进行 seek，即使日志因为某些原因而包含了未写入完整的命令（比如写入时磁盘已满，写入中途停机，等等）， redis-check-aof 工具也可以轻易地修复这种问题。
* Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写：重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。整个重写操作是绝对安全的，因为 Redis在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新AOF文件进行追加操作。
* AOF 文件有序地保存了对数据库执行的所有写入操作，这些写入操作以 Redis 协议的格式保存，因此 AOF 文件的内容非常容易被人读懂，对文件进行分析（parse）也很轻松。 导出（export）AOF 文件也非常简单：举个例子，如果你不小心执行了 FLUSHALL 命令，但只要 AOF 文件未被重写，那么只要停止服务器，移除 AOF 文件末尾的 FLUSHALL 命令，并重启 Redis，就可以将数据集恢复到FLUSHALL执行之前的状态。

### AOF的缺点
* 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
* 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB。在一般情况下，每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。
* AOF 在过去曾经发生过这样的 bug ： 因为个别命令的原因，导致 AOF 文件在重新载入时，无法将数据集恢复成保存时的原样。 （举个例子，阻塞命令 BRPOPLPUSH 就曾经引起过这样的 bug 。） 测试套件里为这种情况添加了测试： 它们会自动生成随机的、复杂的数据集， 并通过重新载入这些数据来确保一切正常。 虽然这种 bug 在 AOF 文件中并不常见， 但是对比来说， RDB 几乎是不可能出现这种 bug 的。

### AOF三种同步策略

##### * always：将缓存区的内容总是即时写到AOF文件中。
##### * everysec：将缓存区的内容每隔一秒写入AOF文件中。
##### * no ：写入AOF文件中的操作由操作系统决定，一般而言为了提高效率，操作系统会等待缓存区被填满，才会开始同步数据到磁盘。

| 命令 | always | everysec | no |
| ------------- | ------------- | ------------- | ------------- |
| 优点  | 不丢失数据  | 只丢一秒数据 | 不用管，操作系统去管 | 
| 缺点  | IO开销大，一般的sata盘只有几百TPS  | 丢了一秒数据 | 不可控，不知道什么时候刷盘，也不知道会丢失多少数据 |

通常使用everysec策略，这也是AOF的默认策略。

### AOF重写
随着时间的推移，命令的逐步写入。AOF文件也会逐渐变大。当我们用AOF来恢复时会很慢，而且当文件无限增大时，对硬盘的管理，对写入的速度也会有产生影响。Redis当然考虑到这个问题，所以就有了AOF重写。
```
原生AOF：
set hello world
set hello java
set hello python
incr counter
incr counter
rpush mylist a
rpush mylist b
rpush mylist c
过期数据
重写后的AOF:
set hello python
set incr 2
rpush mylist a b c
```
AOF重写就是把过期的、没用的、重复的以及可优化的命令，进行化简。只取最终有价值的结果。虽然写入操作很频繁，但系统定义的key的量是相对有限的。
AOF重写可以大大压缩最终日志文件的大小。从而减少磁盘占用量，加快数据恢复速度。比如我们有个计数的服务，有很多自增的操作，比如有一个key自增到1个亿，对AOF文件来说就是一亿次incr。AOF重写就只用记1条记录。

#### * AOF重写两种方式
        * bgrewriteaof命令触发AOF重写
        redis客户端向Redis发bgrewriteaof命令，redis服务端fork一个子进程去完成AOF重写。这里的AOF重写，是将Redis内存中的数据进行一次回溯，回溯成AOF文件。而不是重写AOF文件生成新的AOF文件去替换。
        * AOF自动重写配置
        auto-aof-rewrite-min-size：AOF文件重写需要的尺寸
        auto-aof-rewrite-percentage：AOF文件增长率
        redis提供了aof_current_size和aof_base_size，分别用来统计AOF当前尺寸（单位：字节）和AOF上次启动和重写的尺寸（单位：字节）。
        AOF自动重写的触发时机，同时满足以下两点）：
        aof_current_size > auto-aof-rewrite-min-size
        aof_current_size - aof_base_size/aof_base_size > auto-aof-rewrite-percentage

#### * AOF重写配置
修改配置文件：
```
# 开启正常AOF的append刷盘操作
appendonly yes
# AOF文件名
appendfilename "appendonly-6379.aof"
# 每秒刷盘
appendfsync everysec
# 文件目录
dir /opt/soft/redis/data
# AOF重写增长率
auto-aof-rewrite-percentage 100  (0表示禁用自动重写功能)
# AOF重写最小尺寸
auto-aof-rewrite-min-size 64mb 
# AOF重写期间是否暂停append操作。AOF重写非常消耗磁盘性能，而正常的AOF过程中也会往磁盘刷数据。
# 通常偏向考虑性能，设为yes。万一重写失败了，这期间正常AOF的数据会丢失，因为我们选择了重写期间放弃了正常AOF刷盘。
no-appendfsync-on-rewrite yes
```

#### * AOF运作方式
AOF 重写和 RDB 创建快照一样，都巧妙地利用了写时复制机制。

以下是 AOF 重写的执行步骤：

1、Redis 执行 fork() ，现在同时拥有父进程和子进程。

2、子进程开始将新 AOF 文件的内容写入到临时文件。

3、对于所有新执行的写入命令，父进程一边将它们累积到一个内存缓存中，一边将这些改动追加到现有 AOF 文件的末尾： 这样即使在重写的中途发生停机，现有的 AOF 文件也还是安全的。

4、当子进程完成重写工作时，它给父进程发送一个信号，父进程在接收到信号之后，将内存缓存中的所有数据追加到新 AOF 文件的末尾。

5、搞定！现在 Redis 原子地用新文件替换旧文件，之后所有命令都会直接追加到新 AOF 文件的末尾。


### 实操实验
[AOF触发方式](https://segmentfault.com/a/1190000012316003)


