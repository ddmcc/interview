#### rocketMQ怎么保证消息不丢失？

主要分为3个阶段：

**1.生产阶段**

- **同步发送**：同步发送时只要 `send()` 方法没有抛出异常，就可以认为消息发送成功，即消息队列Broker成功接受到了消息。

- **异步发送**：使用异步发送方式时记得重写 `SendCallback` 类的两个方法，在 `onSuccess()` 方法中更新消息的发送状态为发送成功,只要不发生异常且回调了 `onSuccess()` 方法也可以认为成功发送到了Broker

无论使用同步还是异步的发送方式，都需要判断 `SendStatus` 是不是 `SEND_OK`。但 `SEND_OK` 并不意味着它是可靠的。要确保不会丢失任何消息，还应启用 `SYNC_MASTER` 或 `SYNC_FLUSH`


**2.消息队列Broker存储阶段**

- 刷盘方式：默认的情况下，消息队列为了快速响应，在接受到生产者的请求，将消息保存在内存成功之后，就会立刻返回ACK响应给生产者。所以可以将消息刷盘方式修改为同步刷盘 `flushDiskType=SYNC_FLUSH`

- 集群部署：使用 **多Master多Slave模式-同步双写** 

>## master 节点配置
>brokerRole=SYNC_MASTER
>flushDiskType=SYNC_FLUSH
>
>## slave 节点配置
>brokerRole=SLAVE
>flushDiskType=ASYNC_FLUSH

**3.消费阶段**

消费者拉取消息进行本地业务处理，业务处理完成才能提交ACK，`ConsumeConcurrentlyStatus.CONSUME_SUCCESS`，切不可先提交ACK再进行业务处理。如果业务处理出现异常情况，可以先返回 `ConsumeConcurrentlyStatus.RECONSUME_LATER` 等待消息队列的下次重试


#### RocketMQ如何集群部署？

- 1.单Master模式

一旦Broker重启或者宕机时，会导致整个服务不可用。不建议线上环境使用,可以用于本地测试

- 2.多Master模式

一个集群无Slave，全是Master，例如2个Master或者3个Master，这种模式的优缺点如下：

优点：配置简单，单个Master宕机或重启维护对应用无影响，在磁盘配置为RAID10时，即使机器宕机不可恢复情况下，由于RAID10磁盘非常可靠，消息也不会丢（异步刷盘丢失少量消息，同步刷盘一条不丢），性能最高；

缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可被消费，消息实时性会受到影响

- 3.多Master多Slave模式-异步复制

每个Master配置一个Slave，有多对Master-Slave，HA采用异步复制方式，主备有短暂消息延迟（毫秒级），这种模式的优缺点如下：

优点：即使磁盘损坏，消息丢失的非常少，且消息实时性不会受影响，同时Master宕机后，消费者仍然可以从Slave消费，而且此过程对应用透明，不需要人工干预，性能同多Master模式几乎一样；

缺点：Master宕机，磁盘损坏情况下会丢失少量消息

- 4.多Master多Slave模式-同步双写

每个Master配置一个Slave，有多对Master-Slave，HA采用同步双写方式， **即只有主备都写成功，才向应用返回ACK** ，这种模式的优缺点如下：

优点：数据与服务都无单点故障，Master宕机情况下，消息无延迟，服务可用性与数据可用性都非常高；

缺点：性能比异步复制模式略低（大约低10%左右），发送单个消息的RT会略高，且目前版本在主节点宕机后，备机不能自动切换为主机。

- 5.Dledger 模式

RocketMQ 4.5 以前的版本大多都是采用 `Master-Slave` 架构来部署，能在一定程度上保证数据的不丢失，也能保证一定的可用性。

但是那种方式 的缺陷很明显，最大的问题就是当 Master Broker 挂了之后 ，没办法让 Slave Broker 自动 切换为新的 Master Broker，需要手动更改配置将 Slave Broker 设置为 Master Broker，以及重启机器，这个非常麻烦。
在手式运维的期间，可能会导致系统的不可用。

使用 Dledger 技术要求至少由三个 Broker 组成 ，一个 Master 和两个 Slave，这样三个 Broker 就可以组成一个 Group ，也就是三个 Broker 可以分组来运行。一但 Master 宕机，Dledger 就可以从剩下的两个 Broker 中选举一个 Master 继续对外提供服务

#### RocketMQ消息是推的还是拉的？

RocketMQ实质上还是pull模式，`Consumer` 发送拉取请求到 `Broker` 端，如果 `Broker` 有数据则返回，然后 `Consumer` 端再次拉取。如果 `Broker` 端没有数据，不立即返回，而是等待一段时间（例如30s）。

- 如果在等待的这段时间，有新消息到来，则激活 `consumer` 发送来的请求，立即将消息通过 `channel` 写入，随后 `Consumer` 端再次发起拉取请求
- 如果等待超时（例如30），也会直接返回，不会将这个请求一直hold住，`Consumer` 端再次拉取

长轮询解决轮询带来的频繁请求服务端但是没有的问题一旦新的数据到了，那么消费者能立马就可以获取到新的数据，所以从效果上，有点像是push的感觉


#### RocketMQ的重试机制？

- **默认重试次数：** Product默认是2次，而Consumer默认是16次。同步发送和异步发送，均支持消息发送重试
  - 同步发送：调用线程会一直阻塞，直到某次重试成功或最终重试失败（返回错误码或抛出异常）。
  - 异步发送：调用线程不会阻塞，但调用结果会通过回调的形式，以异常事件或者成功事件返回
- **重试时间间隔：** Product是立刻重试，而Consumer是有一定时间间隔的
- **而对于Consumer在广播情况下重试失效**

消息重试原理：

`RocketMQ` 会为每个消费者组都设置一个 `Topic` 名称为 `%RETRY%+consumerGroup` 的重试队列（这里需要注意的是，这个Topic的重试队列是针对消费组，而不是针对每个Topic设置的），用于暂时保存因为各种异常而导致 `Consumer` 端无法消费的消息。

考虑到异常恢复需要一些时间，`RocketMQ` 会为重试队列设置多个重试级别，每个重试级别都有与之对应的重新投递延时，重试次数越多投递延时就越大。`RocketMQ` 对于重试消息的处理是先保存至 `Topic` 名称为 `SCHEDULE_TOPIC_XXXX` 的延迟队列中，后台定时任务按照对应的时间进行 `Delay` 后重新保存至 `%RETRY%+consumerGroup` 的重试队列中。

正常情况下无法被消费的消息称为死信消息 `(Dead-Letter Message)`，存储死信消息的特殊队列称为死信队列 `(Dead-Letter Queue)`。在 `RocketMQ` 中消息重试超过一定次数后（默认16次）就会被放到死信队列中。可以在控制台 `Topic` 列表中看到“DLQ”相关的Topic，默认命名是：`%DLQ%消费组名称`（死信Topic），死信队列也可以被订阅和消费。
