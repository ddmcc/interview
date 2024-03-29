## Redis

#### Redis常见数据结构

- **`string`**

- **`list`** ：`Redis` 的 `list` 的实现为一个 **双向链表**，即可以支持反向查找和遍历：发布与订阅或者说消息队列

- **`hash`** ：`hash` 类似于 **JDK1.8** 前的 `HashMap`，内部实现也差不多(数组 + 链表)。不过，`Redis` 的 `hash` 做了更多优化。另外，`hash` 是一个 `string` 类型的 field 和 value 的映射表，特别适合用于存储对象，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值

- **`set`** ：`set` 类似于 Java 中的 `HashSet` 。`Redis` 中的 `set` 类型是一种无序集合，集合中的元素没有先后顺序。当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。可以基于 set 轻易实现交集、并集、差集的操作

- **`sorted set`** ：和 `set` 相比，`sorted set` 增加了一个权重参数 `score`，使得集合中的元素能够按 `score` 进行有序排列，还可以通过 `score` 的范围来获取元素的列表

- **`bitmap`** ：bitmap 存储的是连续的二进制数字（0 和 1），通过 bitmap, 只需要一个 bit 位来表示某个元素对应的值或者状态，key 就是对应元素本身 。我们知道 8 个 bit 可以组成一个 byte，所以 bitmap 本身会极大的节省储存空间


#### Redis单线程模型

- Redis 基于 Reactor 模式开发了自己的网络事件处理器：这个处理器被称为文件事件处理器（file event handler）。文件事件处理器使用 I/O 多路复用（multiplexing）程序来同时监听多个套接字，并根据 套接字目前执行的任务来为套接字关联不同的事件处理器。

- 当被监听的套接字准备好执行连接应答（accept）、读取（read）、写入（write）、关 闭（close）等操作时，与操作相对应的文件事件就会产生，这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。

- 虽然文件事件处理器以单线程方式运行，但通过使用 I/O 多路复用程序来监听多个套接字，文件事件处理器既实现了高性能的网络通信模型，又可以很好地与 Redis 服务器中其他同样以单线程方式运行的模块进行对接，这保持了 Redis 内部单线程设计的简单性。


#### Redis没有使用多线程？为什么不使用多线程？

但如果严格来讲从Redis4.0之后并不是单线程，除了主线程外，它也有后台线程在处理一些较为缓慢的操作，例如清理脏数据、无用连接的释放、大 key 的删除等等。大体上来说，Redis 6.0 之前主要还是单线程处理

我觉得主要原因有下面 3 个：

- 单线程编程容易并且更容易维护；
- Redis 的性能瓶颈不再 CPU ，主要在内存和网络；
- 多线程就会存在死锁、线程上下文切换等问题，甚至会影响性能


#### Redis6.0之后为何引入了多线程？

**Redis6.0 引入多线程主要是为了提高网络 IO 读写性能** ，因为这个算是 `Redis` 中的一个性能瓶颈（Redis 的瓶颈主要受限于内存和网络）。

虽然，Redis6.0 引入了多线程，但是 Redis 的多线程只是在网络数据的读写这类耗时操作上使用了， 执行命令仍然是单线程顺序执行。因此，你也不需要担心线程安全问题。
Redis6.0 的多线程默认是禁用的


#### 为什么要用Redis、为什么要用缓存？

**高性能：**
保证用户下一次再访问这些数据的时候就可以直接从缓存中获取了。操作缓存就是直接操作内存，所以速度相当快。不过，要保持数据库和缓存中的数据的一致性。 如果数据库中的对应数据改变的之后，同步改变缓存中相应的数据即可！
    
**高并发：**

直接操作缓存能够承受的数据库请求数量是远远大于直接访问数据库的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。进而，我们也就提高的系统整体的并发


#### Redis是如何判断数据是否过期的呢？

`Redis` 通过一个叫做 **过期字典**（可以看作是 `hash` 表）来保存数据过期的时间。过期字典的键指向 `Redis` 数据库中的某个 key(键)，过期字典的值是一个 long long 类型的整数，这个整数保存了 `key` 所指向的数据库键的过期时间（毫秒精度的 UNIX 时间戳）


#### 过期的数据的删除策略了解么？

- **惰性删除** ：只会在取出 key 的时候才对数据进行过期检查。这样对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。

- **定期删除** ： 每隔一段时间随机抽取一批（20个） key 执行删除过期 key 操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。


定期删除对内存更加友好，惰性删除对 CPU 更加友好。两者各有千秋，所以 Redis 采用的是 定期删除+惰性/懒汉式删除

#### Redis的内存淘汰机制?

- **noeviction:** 当内存不足以容纳新写入数据时，新写入操作会报错。
- **allkeys-lru：** 当内存不足以容纳新写入数据时，在键空间（server.db[i].dict）中，移除最近最少使用的 key（这个是最常用的）。
- **allkeys-random：** 当内存不足以容纳新写入数据时，在键空间（server.db[i].dict）中，随机移除某个 key。
- **volatile-lru：** 当内存不足以容纳新写入数据时，在设置了过期时间的键空间（server.db[i].expires）中，移除最近最少使用的 key。
- **volatile-random：** 当内存不足以容纳新写入数据时，在设置了过期时间的键空间（server.db[i].expires）中，随机移除某个 key。
- **volatile-ttl：** 当内存不足以容纳新写入数据时，在设置了过期时间的键空间（server.db[i].expires）中，有更早过期时间的 key 优先移除。

>在配置文件中，通过maxmemory-policy可以配置要使用哪一个淘汰机制，默认为：noeviction。
>其中 allkeys-xxx 表示从所有的键值中淘汰数据，而 volatile-xxx 表示从设置了过期键的键值中淘汰数据
 

#### Redis持久化机制

**Redis 的一种持久化方式叫快照（snapshotting，RDB），另一种方式是只追加文件（append-only file, AOF）**

- **`RDB（默认）`**

Redis 可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。Redis 创建快照之后，可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本（Redis 主从结构，主要用来提高 Redis 性能），还可以将快照留在原地以便重启服务器的时候使用

> RDB配置方式：save `<seconds>` `<changes>`
>
>其中，`<seconds>` 表示每隔多少秒执行一次快照，`<changes>` 表示修改了多少次数据后执行快照
>
>默认配置为：
> 
>save 900 1
>
>save 300 10
>
>save 60 10000
>
>表示 900秒内如果有1次/300秒内有10次/60秒内有10000次的变更，则创建快照
    
- **`AOP`**

与快照持久化相比，`AOF` 持久化 的实时性更好，因此已成为主流的持久化方案。默认情况下 Redis 没有开启 AOF（append only file）方式的持久化 开启 AOF 持久化后每执行一条会更改 Redis 中的数据的命令，Redis 就会将该命令写入硬盘中的 AOF 文件
    
>appendfsync always    #每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度
>
>appendfsync everysec  #每秒钟同步一次，显示地将多个写命令同步到硬盘
>
>appendfsync no        #让操作系统决定何时进行同步      

让 `Redis` 每秒同步一次 `AOF` 文件，`Redis` 性能几乎没受到任何影响。而且这样即使出现系统崩溃，用户最多只会丢失一秒之内产生的数据。当硬盘忙于执行写入操作的时候，`Redis` 还会优雅的放慢自己的速度以便适应硬盘的最大写入速度。


#### Redis事务

使用 `MULT` 命令后可以输入多个命令。`Redis` 不会立即执行这些命令，而是将它们放到队列，当调用了EXEC命令将执行所有命令.你也可以通过 `DISCARD`  命令取消一个事务，它会清空事务队列中保存的所有命令
Redis 是不支持 `roll back` 的

#### 什么是缓存击穿？

我们的业务通常会有几个数据会被频繁地访问，比如秒杀活动，这类被频地访问的数据被称为热点数据。

如果缓存中的某个热点数据过期了，此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库很容易就被高并发的请求冲垮，这就是缓存击穿的问题

可以发现缓存击穿跟缓存雪崩很相似，你可以认为缓存击穿是缓存雪崩的一个子集。 应对缓存击穿可以采取前面说到两种方案：

- 互斥锁方案（Redis 中使用 setNX 方法设置一个状态位，表示这是一种锁定状态），保证同一时间只有一个业务线程请求缓存，未能获取互斥锁的请求，要么等待锁释放后重新读取缓存，要么就返回空值或者默认值。
- 不给热点数据设置过期时间，由后台异步更新缓存，或者在热点数据准备要过期前，提前通知后台线程更新缓存以及重新设置过期时间；

#### 什么是缓存穿透？

缓存穿透说简单点就是大量请求的 key 根本不存在于缓存中，也不在数据库中，导致请求直接到了数据库上，根本没有经过缓存这一层

**有哪些解决办法？**

- **缓存无效 key** ：这种方式可以解决请求的 key 变化不频繁的情况，如果黑客恶意攻击，每次构建不同的请求 key，会导致 Redis 中缓存大量无效的 key 
    
- **`布隆过滤器`**
        
通过它我们可以非常方便地判断一个给定数据是否存在于海量数据中。我们需要的就是判断 `key` 是否合法
        
**具体是这样做的** ：把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会走下面的流程
            
1）我们先来看一下，**当一个元素加入布隆过滤器中的时候，会进行哪些操作：**

- 使用布隆过滤器中的哈希函数对元素值进行计算，得到哈希值（有几个哈希函数得到几个哈希值）。

- 根据得到的哈希值，在位数组中把对应下标的值置为 1。


2）我们再来看一下，**当我们需要判断一个元素是否存在于布隆过滤器的时候，会进行哪些操作：**

- 对给定元素再次进行相同的哈希计算；

- 得到值之后判断位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

然后，一定会出现这样一种情况：不同的字符串可能哈希出来的位置相同。 （可以适当增加位数组大小或者调整我们的哈希函数来降低概率）


#### 什么是缓存雪崩？

**缓存在同一时间大面积的失效，后面的请求都直接落到了数据库上，造成数据库短时间内承受大量请求(`缓存服务器奔溃`)，有一些被大量访问数据（`热点缓存`）在某一时刻大面积失效，导致对应的请求直接落到了数据库上**

- 针对 Redis 服务不可用的情况：
  - 采用 Redis 集群，避免单机出现问题整个缓存服务都没办法使用。
  - 限流，避免同时处理大量的请求。

- 针对热点缓存失效的情况：
  - 设置不同的失效时间比如随机设置缓存的失效时间。
  - 缓存永不失效。
  
  
#### 缓存一致性

一般会采用先删除缓存再更新数据库或先更新数据库再删除缓存，

- **第一种先删除在更新：** 你删除了缓存，这时有读的请求发现没缓存，去数据库查询后放入缓存，这时数据库在被更新，导致缓存和数据库不一致。这种情况可以使用双删的策略，就是一开始先删除一次，完后执行完业务，在删除一次。还有就是先更新数据库在删除缓存，这种好处就是保证我更新完缓存也是失效的

- **第二种先更新数据库，再删缓存**

    一般是推荐这种，首先因为删除缓存比更新数据库快的多，这段时间内会读到脏数据的可能性就越小，当然也是存在产生数据不一致的可能性的，比如读操作比写操作还慢，写完数据删除缓存都返回了，读请求才把读出来的脏数据写入缓存。不过这种可能性比较小
不管是先删缓存再更新数据库还是先更新数据库再删缓存，如果删除缓存失败了都会导致缓存跟数据不一致问题！

**保证删除缓存操作的可靠性**

- 消息队列，确保消息删除
    
- 专门程序+消息队列 确保消息删除


#### redis有哪些集群模式？

- **主从复制模式：** 主从复制模式中包含一个主数据库实例（master）与一个或多个从数据库实例（slave）

>具体工作机制为：
 
> 1.slave启动后，向master发送SYNC命令，master接收到SYNC命令后通过bgsave保存快照（即上文所介绍的RDB持久化），并使用缓冲区记录保存快照这段时间内执行的写命令
>
> 2.master将保存的快照文件发送给slave，并继续记录执行的写命令
> 
> 3.slave接收到快照文件后，加载快照文件，载入数据
> 
> 4.master快照发送完后开始向slave发送缓冲区的写命令，slave接收命令并执行，完成复制初始化
>
>此后master每次执行一个写命令都会同步发送给slave，保持master与slave之间数据的一致性

- **Sentinel（哨兵）模式：** 

哨兵顾名思义，就是来为Redis集群站哨的，一旦发现问题能做出相应的应对处理。其功能包括

- 监控master、slave是否正常运行
- 当master出现故障时，能自动将一个slave转换为master（大哥挂了，选一个小弟上位）
- 多个哨兵可以监控同一个Redis，哨兵之间也会自动监控

**哨兵模式的具体工作机制：**

在配置文件中通过 sentinel monitor <master-name> <ip> <redis-port> <quorum> 来定位master的IP、端口，一个哨兵可以监控多个master数据库，只需要提供多个该配置项即可。哨兵启动后，会与要监控的master建立两条连接：

- 一条连接用来订阅master的_sentinel_:hello频道与获取其他监控该master的哨兵节点信息
- 另一条连接定期向master发送INFO等命令获取master本身的信息

与master建立连接后，哨兵会执行三个操作：

- 定期（一般10s一次，当master被标记为主观下线时，改为1s一次）向master和slave发送INFO命令
- 定期向master和slave的_sentinel_:hello频道发送自己的信息
- 定期（1s一次）向master、slave和其他哨兵发送PING命令

发送INFO命令可以获取当前数据库的相关信息从而实现新节点的自动发现。所以说哨兵只需要配置master数据库信息就可以自动发现其slave信息。获取到slave信息后，哨兵也会与slave建立两条连接执行监控。通过INFO命令，哨兵可以获取主从数据库的最新信息，并进行相应的操作，比如角色变更等。

接下来哨兵向主从数据库的_sentinel_:hello频道发送信息与同样监控这些数据库的哨兵共享自己的信息，发送内容为哨兵的ip端口、运行id、配置版本、master名字、master的ip端口还有master的配置版本。这些信息有以下用处：

- 其他哨兵可以通过该信息判断发送者是否是新发现的哨兵，如果是的话会创建一个到该哨兵的连接用于发送PING命令。
- 其他哨兵通过该信息可以判断master的版本，如果该版本高于直接记录的版本，将会更新
- 当实现了自动发现slave和其他哨兵节点后，哨兵就可以通过定期发送PING命令定时监控这些数据库和节点有没有停止服务。

如果被PING的数据库或者节点超时（通过 sentinel down-after-milliseconds master-name milliseconds 配置）未回复，哨兵认为其主观下线（sdown，s就是Subjectively —— 主观地）。如果下线的是master，哨兵会向其它哨兵发送命令询问它们是否也认为该master主观下线，如果达到一定数目（即配置文件中的quorum）投票，哨兵会认为该master已经客观下线（odown，o就是Objectively —— 客观地），并选举领头的哨兵节点对主从系统发起故障恢复。若没有足够的sentinel进程同意master下线，master的客观下线状态会被移除，若master重新向sentinel进程发送的PING命令返回有效回复，master的主观下线状态就会被移除

哨兵认为master客观下线后，故障恢复的操作需要由选举的领头哨兵来执行，选举采用Raft算法：

- 发现master下线的哨兵节点（我们称他为A）向每个哨兵发送命令，要求对方选自己为领头哨兵
- 如果目标哨兵节点没有选过其他人，则会同意选举A为领头哨兵
- 如果有超过一半的哨兵同意选举A为领头，则A当选
- 如果有多个哨兵节点同时参选领头，此时有可能存在一轮投票无竞选者胜出，此时每个参选的节点等待一个随机时间后再次发起参选请求，进行下一轮投票竞选，直至选举出领头哨兵

选出领头哨兵后，领头者开始对系统进行故障恢复，从出现故障的master的从数据库中挑选一个来当选新的master,选择规则如下：

- 所有在线的slave中选择优先级最高的，优先级可以通过slave-priority配置
- 如果有多个最高优先级的slave，则选取复制偏移量最大（即复制越完整）的当选
- 如果以上条件都一样，选取id最小的slave

挑选出需要继任的slave后，领头哨兵向该数据库发送命令使其升格为master，然后再向其他slave发送命令接受新的master，最后更新数据。将已经停止的旧的master更新为新的master的从数据库，使其恢复服务后以slave的身份继续运行

- **Cluster模式：**

Cluster采用无中心结构,它的特点如下：

- 所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽
- 节点的fail是通过集群中超过半数的节点检测失效时才生效
- 客户端与redis节点直连,不需要中间代理层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可

Cluster模式的具体工作机制：

- 在Redis的每个节点上，都有一个插槽（slot），取值范围为0-16383
- 当我们存取key的时候，Redis会根据CRC16的算法得出一个结果，然后把结果对16384求余数，这样每个key都会对应一个编号在0-16383之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作
- 为了保证高可用，Cluster模式也引入主从复制模式，一个主节点对应一个或者多个从节点，当主节点宕机的时候，就会启用从节点
- 当其它主节点ping一个主节点A时，如果半数以上的主节点与A通信超时，那么认为主节点A宕机了。如果主节点A和它的从节点都宕机了，那么该集群就无法再提供服务了

Cluster模式集群节点最小配置6个节点(3主3从，因为需要半数以上)，其中主节点提供读写操作，从节点作为备用节点，不提供请求，只作为故障转移使用