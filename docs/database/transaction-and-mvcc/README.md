### 事务与MVCC

#### 事务的四种隔离级别

- **未提交读(Read Uncommitted)** ：允许脏读，也就是可能读取到其他会话中未提交事务修改的数据，**任何操作都不会加锁**

- **提交读(Read Committed)** ：只能读取到已经提交的数据。Oracle等多数数据库默认都是该级别 (不重复读)，**RC级别中，数据的读取都是不加锁的（采用一致性非锁定读），但是数据的写入、修改和删除是需要加锁的（仅使用Record Lock）**

- **可重复读(Repeated Read)** ：可重复读。在同一个事务内的查询都是事务开始时刻一致的，InnoDB默认级别。在SQL标准中，该隔离级别消除了不可重复读，~~但是还存在幻象读~~（RR级别中通过Next-Key Lock和MVCC解决了幻读），**在 RR 隔离级别下，对于读 `InnoDB` 存储引擎同样使用 一致性非锁定读 ，但加锁上却和RC不同，其使用 `Next-Key Lock`**

- **串行读(Serializable)** ：完全串行化的读，每次读都需要获得表级共享锁，读写相互都会阻塞


#### InnoDB如何实现事务的ACID的？

- MySQL `InnoDB` 引擎使用 `redo log(重做日志)` 保证事务的持久性；

- 使用 `undo log(回滚日志)` 来保证事务的原子性；

- MySQL `InnoDB` 引擎通过 `锁机制`、`MVCC` 等手段来保证事务的隔离性（ 默认支持的隔离级别是 REPEATABLE-READ ）。

- 保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障


#### MVCC在InnoDB中的实现

`MVCC` 的实现依赖于：**隐藏字段、Read View、undo log**。在内部实现中，`InnoDB` 通过数据行的 `DB_TRX_ID` 和 `Read View` 来判断数据的可见性，如不可见，则通过数据行的 `DB_ROLL_PTR` 找到 `undo log` 中的历史版本。每个事务读到的数据版本可能是不一样的，在同一个事务中，用户只能看到该事务创建 `Read View` 之前已经提交的修改和该事务本身做的修改



#### 隐藏字段

在内部，`InnoDB` 存储引擎为每行数据添加了三个 [隐藏字段](https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html)：

- `DB_TRX_ID（6字节）`：表示最后一次插入或更新该行的事务id。此外，`delete` 操作在内部被视为更新，只不过会在记录头 `Record header` 中的 `deleted_flag` 字段将其标记为已删除
- `DB_ROLL_PTR（7字节）` 回滚指针，指向该行的 `undo log` 。如果该行未被更新，则为空
- `DB_ROW_ID（6字节）`：如果没有设置主键且该表没有唯一非空索引时，`InnoDB` 会使用该id来生成聚簇索引



#### ReadView

[`Read View`](https://github.com/facebook/mysql-8.0/blob/8.0/storage/innobase/include/read0types.h#L298) 主要是用来做可见性判断，里面保存了 “当前对本事务不可见的其他活跃事务”

主要有以下字段：

- `m_low_limit_id`：目前出现过的最大的事务ID+1，即下一个将被分配的事务ID。大于这个ID的数据版本均不可见
- `m_up_limit_id`：活跃事务列表 `m_ids` 中最小的事务ID，如果 `m_ids` 为空，则 `m_up_limit_id` 为 `m_low_limit_id`。小于这个ID的数据版本均可见
- `m_ids`：`Read View` 创建时其他未提交的活跃事务ID列表。创建 `Read View `时，将当前未提交事务ID记录下来，后续即使它们修改了记录行的值，对于当前事务也是不可见的。`m_ids` 不包括当前事务自己和已提交的事务（正在内存中）
- `m_creator_trx_id`：创建该 `Read View` 的事务ID



#### undo-log

`undo log` 主要有两个作用：

- 当事务回滚时用于将数据恢复到修改前的样子
- 另一个作用是 `MVCC` ，当读取记录时，若该记录被其他事务占用或当前版本对该事务不可见，则可以通过 `undo log` 读取之前的版本数据，以此实现非锁定读



**在 `InnoDB` 存储引擎中 `undo log` 分为两种： `insert undo log` 和 `update undo log`：**



1. **`insert undo log`** ：指在 `insert` 操作中产生的 `undo log`。因为 `insert` 操作的记录只对事务本身可见，对其他事务不可见，故该 `undo log` 可以在事务提交后直接删除。不需要进行 `purge` 操作



**`insert` 时的数据初始状态：**

![markdown](https://ddmcc-1255635056.file.myqcloud.com/317e91e1-1ee1-42ad-9412-9098d5c6a9ad.png)

2. **`update undo log`** ：`update` 或 `delete` 操作中产生的 `undo log`。该 `undo log`可能需要提供 `MVCC` 机制，因此不能在事务提交时就进行删除。提交时放入 `undo log` 链表，等待 `purge线程` 进行最后的删除



**数据第一次被修改时：**



![markdown](https://ddmcc-1255635056.file.myqcloud.com/c52ff79f-10e6-46cb-b5d4-3c9cbcc1934a.png)

**数据第二次被修改时：**



![markdown](https://ddmcc-1255635056.file.myqcloud.com/6a276e7a-b0da-4c7b-bdf7-c0c7b7b3b31c.png)

不同事务或者相同事务的对同一记录行的修改，会使该记录行的 `undo log` 成为一条链表，链首就是最新的记录，链尾就是最早的旧记录



#### 数据可见性算法

在 `InnoDB` 存储引擎中，创建一个新事务后，执行每个 `select` 语句前，都会创建一个快照（Read View），**快照中保存了当前数据库系统中正处于活跃（没有commit）的事务的ID号**。其实简单的说保存的是系统中当前不应该被本事务看到的其他事务ID列表（即m_ids）。当用户在这个事务中要读取某个记录行的时候，`InnoDB` 会将该记录行的 `DB_TRX_ID` 与 `Read View` 中的一些变量及当前事务ID进行比较，判断是否满足可见性条件

[具体的比较算法](https://github.com/facebook/mysql-8.0/blob/8.0/storage/innobase/include/read0types.h#L161)如下：[图源](https://leviathan.vip/2019/03/20/InnoDB%E7%9A%84%E4%BA%8B%E5%8A%A1%E5%88%86%E6%9E%90-MVCC/#MVCC-1)

![markdown](https://ddmcc-1255635056.file.myqcloud.com/8778836b-34a8-480b-b8c7-654fe207a8c2.png)

1. 如果记录 DB_TRX_ID < m_up_limit_id，那么表明最新修改该行的事务（DB_TRX_ID）在当前事务创建快照之前就提交了，所以该记录行的值对当前事务是可见的

2. 如果 DB_TRX_ID >= m_low_limit_id，那么表明最新修改该行的事务（DB_TRX_ID）在当前事务创建快照之后才修改该行，所以该记录行的值对当前事务不可见。跳到步骤5

3. m_ids 为空，则表明在当前事务创建快照之前，修改该行的事务就已经提交了，所以该记录行的值对当前事务是可见的

4. 如果 m_up_limit_id <= DB_TRX_ID < m_up_limit_id，表明最新修改该行的事务（DB_TRX_ID）在当前事务创建快照的时候可能处于“活动状态”或者“已提交状态”；所以就要对活跃事务列表 m_ids 进行查找（源码中是用的二分查找，因为是有序的）

    - 如果在活跃事务列表 m_ids 中能找到 DB_TRX_ID，表明：①在当前事务创建快照前，该记录行的值被事务ID为 DB_TRX_ID 的事务修改了，但没有提交；或者 ②在当前事务创建快照后，该记录行的值被事务ID为 DB_TRX_ID 的事务修改了。这些情况下，这个记录行的值对当前事务都是不可见的。跳到步骤5

    - 在活跃事务列表中找不到，则表明“id为trx_id的事务”在修改“该记录行的值”后，在“当前事务”创建快照前就已经提交了，所以记录行对当前事务可见

5. 在该记录行的 DB_ROLL_PTR 指针所指向的 `undo log` 取出快照记录，用快照记录的 DB_TRX_ID 跳到步骤1重新开始判断，直到找到满足的快照版本或返回空


#### 在RC和RR隔离级别下MVCC的差异？

在RC和RR级别下，`InnoDB` 都会使用 **一致性非锁定读（MVCC）** ，区别是快照的获取时机不同：

- **RC级别**：在 `Read Committed` 级别下，事务中 **`每一次`** 一致性读取（普通select）都会获取新的快照（ReadView）


- **RR级别**：如果事务隔离级别为 `Repeatable Read`（默认级别），只有在事务开始之后 **`第一次`** 执行一致性读取（普通select）才会去获取快照（ReadView），并且后面每次查询都会以这份快照为准，直至事务结束


#### RR级别下会发生幻读吗？为什么？

在 `Repeatable Read` 级别下不会发生幻读。在执行普通 `select` 语句时，使用的是 **一致性非锁定读（MVCC、快照读）** ，会根据第一次查询生成的活跃事务ID列表来判断数据可见性。
所以在第一次查询之后变更的数据是不可见的，解决了一致性非锁定读下的幻读问题

`MVCC` 解决了快照读下的幻读问题，但是对于 **锁定读**，每一次读取即使在同一事务中，也会设置和读取自己的新快照，所以读取的都是最新的数据。这样就会产生幻读的问题

`InnoDB` 使用 `Next-Key Lock` 来锁住记录本身和前后间隙，使其它事务不能插入数据，来解决幻读的问题 
