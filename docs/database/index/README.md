### 索引


#### 索引有什么用？有什么缺点？

**索引是一种用于快速查询和检索数据的数据结构。常见的索引结构有: B 树， B+树和 Hash**

索引的作用就相当于目录的作用。打个比方: 我们在查字典的时候，如果没有目录，那我们就只能一页一页的去找我们需要查的那个字，速度很慢。如果有目录了，我们只需要先去目录里查找字的位置，然后直接翻到那一页就行了


##### 优点： 

- 使用索引可以大大加快 数据的检索速度（大大减少的检索的数据量）, 这也是创建索引的最主要的原因。

- 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。

#### 缺点 ：

- 创建索引和维护索引需要耗费许多时间。当对表中的数据进行增删改的时候，如果数据有索引，那么索引也需要动态的修改，会降低 SQL 执行效率。

- 索引需要使用物理文件存储，也会耗费一定空间。

但是，**使用索引一定能提高查询性能吗?**

大多数情况下，索引查询都是比全表扫描要快的。但是如果数据库的数据量不大，那么使用索引也不一定能够带来很大提升


#### 介绍一下覆盖索引？什么是回表？

索引分为聚簇索引和二级索引，聚簇索引子节点为主键，叶子节点存的是数据行。二级索引子节点存的是索引键、叶子节点存的是主键

通常如果使用的是二级索引，会先在二级索引中查询符合的主键，然后再用主键去聚簇索引中查询其它列的数据或过滤，**这个过程称为回表**

但是如果查询的列和条件列都在二级索引上，那么可以直接提供查询结果，而不需要查询聚簇索引中的记录（mysql5.0或以下版本不支持），则被称为 **覆盖索引**


#### 索引的底层数据结构

##### Hash索引

哈希表是键值对的集合，通过键(key)即可快速取出对应的值(value)，因此哈希表可以快速检索数据（接近 O（1））。

通过哈希算法，我们可以快速找到 `value` 对应的 `index`，找到了 `index` 也就找到了对应的 `value`。但哈希算法有 `Hash` 冲突问题，也就是说多个不同的 `key` 最后得到的 `index` 相同。通常情况下，我们常用的解决办法是 **链地址法**。链地址法就是将哈希冲突数据存放在链表中

既然哈希表这么快，为什么MySQL **没有使用其作为索引的数据结构呢？**

1. **Hash 冲突问题** ：我们上面也提到过Hash 冲突了，不过对于数据库来说这还不算最大的缺点。

2. **Hash 索引不支持顺序和范围查询** (Hash 索引不支持顺序和范围查询是它最大的缺点： 假如我们要对表中的数据进行排序或者进行范围查询，那 Hash 索引可就不行了


##### **`B+ tree索引`**

在 `InnoDB` 中，**每一个索引都会单独维护一棵索引树** ，又分为聚簇索引和二级索引

`聚簇索引` 是按照表 `主键` 构造的，树中同时保存了索引和数据行，数据行被存放在索引的叶子节点中。也将聚簇索引的叶子节点称为数据页。因为无法同时把数据行存放在两个不同的地方，所以一个表只能有一个 `聚簇索引`。 **如果没有定义主键，`InnoDB` 会选择一个唯一非空索引字段代替，如果没有这样的索引，那么 `InnoDB`会隐式定义一个 `6` 字节的 `rowId` 来作为 `聚簇索引`**

除了聚簇索引其它的都是二级索引，和聚簇索引不同的是：二级索引叶子节点存的是主键的值，非叶子节不存储数据行，只存储指向下层叶子的指针和索引键值的虚记录

> 虚记录：并不存在于数据表中的记录


为了减少IO次数，存储引擎会整页整页的读取索引记录，并且索引页都是固定大小的，对于 `InnoDB` 默认为 `16kb`。这也是 `索引字段越小越好` 的原因：因为磁盘块的大小也就是一个数据页的大小，
是固定的（默认16k），那么在总数据量固定的情况下，如果数据项占的空间越小，则每页能存的数据项数量越多，那么树的高度越低。这也是为什么 `b+` 树要求把真实的数据放到叶子节点而不是内层节点，一旦放到内层节点，每个磁盘块的数据项会大幅度下降，导致树增高


索引页之间通过双向链表链接（Page Header中的PAGE_PREV和PAGE_NEXT记录上、下页的位置），页按照索引键的顺序排序；另外每个页中的记录也是通过单向链表进行维护的（Recorder Header的最后两个字节记录下一行偏移量）。**按照索引键排序的好处就是对于索引键的排序查找和范围查找非常快**，只需要查找对应符合的值，如果判断不满足，可以立即结束查询，不需要继续查找。如果没有排好序的话，则需要全表遍历

在索引页内部记录查找中，使用的类似 `跳表` 查找，在定位到页后，会先利用页目录（Page Directory）来进行 `二分查找`，定位到距离数据较近的槽点（Slot），然后再遍历链表


#### B树和B➕树两者有何异同呢？

- `B` 树的所有节点既存放键(key) 也存放 数据(data)；而 B+树只有叶子节点存放 `key` 和 `data`，其他内节点只存放 `key`

- `B` 树的叶子节点都是独立的; B+树的叶子节点有一条引用链指向与它相邻的叶子节点

- `B` 树的检索的过程相当于对范围内的每个节点的关键字做二分查找，可能还没有到达叶子节点，检索就结束了；而 B+树的检索效率就很稳定了，任何查找都是从根节点到叶子节点的过程，叶子节点的顺序检索很明显。


#### 为什么用b➕tree来做索引？

- **减少IO次数**：`B+Tree` 树非叶子节点只存储 `key` 值，因此相对于 `B-Tree` 节点可以存储更多的数据，每次读入内存的 `key` 值就更多，相对来说 `I/O` 次数就降低

- **查询效率稳定**：任何数据的查找都是必须从叶子节点到非叶子节点，所以说每个数据查找的效率几乎都是相同的

- **索引/数据有序**：只需要遍历叶子节点就可以对所有的 `key` 进行扫描，有序在范围查找时和排序查找时效率更高


#### 为什么索引列越小越好？

因为索引页是固定大小的，默认为16k，索引列越小每页能放的记录就越多，这样每次IO读取的就越多，可以减少IO次数


#### 为什么建议主键越小越好？

- 在二级索引中，叶子节点存储的是主键，如果主键越小，那么索引所占用的空间也就会越小

- 在聚簇索引中，主键作为索引的key，如果key越小每页能放的记录就越多，这样每次IO读取的就越多，可以减少IO次数


#### 说说页合并和页分裂？

##### 页分裂

`B+` 树为了维护索引有序性，在插入新值的时候需要做必要的维护。如果插入的位置在页中间，则需要 **`逻辑上`** 挪动前后的数据（改变指向下一数据行的指针）。更糟糕的情况是，
如果所在数据页的数据已经满了，根据 `B+` 树的算法需要新建一个新的页，然后挪动部分数据过去并在 `逻辑上` 挪动相关页的顺序，这个过程中性能肯定会受到影响

除了性能外，页分裂操作还影响数据页的利用率。原本放在一个页的数据，现在分到两个页中，整体空间利用率降低大约 `50%`。**页分裂发生在插入数据或更新数据**


##### 页合并

**删除记录时，不会实际删除该记录。相反，它将记录标记为已删除，并且它所使用的空间可以被其它记录声明使用**。当页中删除的记录达到 `MERGE_THRESHOLD`（默认页体积的50%），InnoDB会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用（被合并页体积也需要小于50%）,
合并后，相邻的页变成一个空页，可以接纳新数据

当我们进行UPDATE操作，并且使页中记录数量低于阈值时，InnoDB也会进行一样的操作

规则是：**页合并发生在删除或更新操作中，关联到当前页的相邻页**

另一方面，要记住在合并和分裂的过程，InnoDB会在索引树上加写锁（x-latch）。在操作频繁的系统中这可能会是个隐患。它可能会导致索引的锁争用（index latch contention）。如果表中没有合并和分裂（也就是写操作）的操作，称为“乐观”更新，只需要使用读锁（S）。带有合并也分裂操作则称为“悲观”更新，使用写锁（X）

#### 为什么通常建议自增列作为主键？

##### `性能上`

用自增列作为主键 ，这样可以保证数据行是按顺序写入的，每次插入都是追加操作，当达到页的最大填充因子时（InnoDB默认最大填充因子是页大小的 15/16，留出部分空间用于后续修改），
下一条记录就会写入新的页中。不涉及到挪动其它记录，也不会触发叶子节点的分裂

而有业务逻辑的字段做主键或者使用UUID，则不能保证有序插入，这样写入数据的成本相对较高。**总结以下是一些缺点：**

- 写入目标页可能已经刷到磁盘上并从缓存中清除，或者是还没有被加载到缓存中，InnoDB在插入之前不得不先找到并从磁盘读取目标页到内存。这将导致大量的随机I/O

- 因为写入是乱序的，InnoDB不得不频繁地做页分裂操作，以便为新的行分配空间，页分裂会导致移动大量的数据，一次插入最少需要修改三个页而不是一个页

- 由于频繁的页分裂，页会变得稀疏并被不规则的填充，最终数据会有碎片

##### `空间上`

而且通常业务主键都比较大，比如用身份证号做主键，那么每个二级索引的叶子节点都需要存储，这也大大的增加了索引的空间占用。而如果使用整型做主键，则只需4个字节，长整型（bigint）也只需8字节。所以，从性能和存储空间方面考量，自增主键往往是更合理的选择


#### 建立索引的一些原则

##### **`最左前缀匹配原则`**

- 如果不是按照索引的最左列开始查找，则无法使用索引

- 不能跳过索引中的列

- 如果查询中有某个列是范围查询，则其右边所有列都无法使用索引


##### **`尽量选择区分度高的列作为索引`**

区分度的公式是 `count(distinct col) / count(*)`，表示字段不重复的比例，比例越大查询效率越高


##### **`尽量的扩展索引，不要新建索引`**

- 空间：InnoDB 会为每个索引都建立 B+ 树索引，所以会占用更多的空间

- 性能：每次修改数据时要对索引进行维护，多棵索引树无疑增加了维护成本。并且在查询上，多列索引有机会使用 覆盖索引 和 索引下推 来提升查询效率


#### 使用索引的注意事项

##### **索引失效情况**

- 索引列不能参与计算：比如 from_unixtime(create_time) = ’2014-05-29’ 或 left(code, 6) = ‘010108’ 或 score + 1 = 80 就不能使用到索引。所以语句应该写成create_time = unix_timestamp(’2014-05-29’) 和 code LIKE ‘010108%’

- 索引列与参数类型不匹配：比如字符串字段索引 phone = 13024532432

- 最左前缀匹配：比如左模糊查询


##### 被频繁更新的字段应该慎重建立索引

虽然索引能带来查询上的效率，但是每个索引都会单独维护索引树，维护索引的成本也是不小的。 如果一个字段不被经常查询，反而被经常修改，那么就更不应该在这种字段上建立索引了
