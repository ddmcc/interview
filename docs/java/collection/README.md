## 集合容器

[ArrayList 源码+扩容机制分析](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/ArrayList%E6%BA%90%E7%A0%81+%E6%89%A9%E5%AE%B9%E6%9C%BA%E5%88%B6%E5%88%86%E6%9E%90) 

[LinkedList 源码](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/LinkedList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90) 

[HashMap(JDK1.8)源码+底层数据结构分析](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap(JDK1.8)%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90) 

[ConcurrentHashMap 源码+底层数据结构分析](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/ConcurrentHashMap%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90)


#### 集合框架底层数据结构总结

- **`List`**
  - ArrayList： 底层Object[]数组，线程不安全，每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity 为偶数就是 1.5 倍，否则是 1.5 倍左右）
  - Vector：底层Object[]数组，线程不安全
  - LinkedList： 底层双向链表(JDK1.6 之前为循环链表，JDK1.7 取消了循环)

- **`Map`**
  - HashMap： JDK1.8 之前 HashMap 由`数组` + `链表` 组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间
  - LinkedHashMap： `LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由 **数组和链表或红黑树** 组成。另外，`LinkedHashMap` 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑
  - HashTable： 数组 + 链表组成的，数组是 HashTable 的主体，链表则是主要为了解决哈希冲突而存在的
  - TreeMap： 红黑树（自平衡的排序二叉树）

- **`Set`**
  - HashSet（无序，唯一）: 基于 `HashMap` 实现的，底层采用 `HashMap` 来保存元素
  - LinkedHashSet：LinkedHashSet 是 HashSet 的子类，并且其内部是通过 LinkedHashMap 来实现的
  - TreeSet（有序，唯一）： 红黑树(自平衡的排序二叉树)


#### 使用什么集合你是怎么选择的？

主要根据集合的特点来选用，比如我们需要根据 `键值` 获取到元素值时就选用 `Map` 接口下的集合，需要自定义排序时选择 `TreeMap`，需要按插入顺序选择 `LinkedHashMap`，不需要排序时就选择 `HashMap`，需要保证线程安全就选用 `ConcurrentHashMap`。

当我们只需要存放元素值时，就选择实现 `Collection` 接口的集合，需要保证元素唯一时选择实现 `Set` 接口的集合比如 `TreeSet` 或 `HashSet`，不需要就选择实现 `List` 接口的比如 `ArrayList` 或 `LinkedList`，然后再根据实现这些接口的集合的特点来选用


#### 为什么要使用集合？而不是数组

当我们需要保存一组 `类型相同的数据` 的时候，我们应该是用一个容器来保存，这个容器就是数组，但是，使用数组存储对象具有一定的弊端， 因为我们在实际开发中，存储的数据的类型是多种多样的，于是，就出现了“集合”，集合同样也是用来存储多个数据的。

**数组的缺点** 是一旦声明之后，长度就不可变了；同时，声明数组时的数据类型也决定了该数组存储的数据的类型；而且，数组存储的数据是 `有序的`、`可重复的`，`特点单一`。 但是集合提高了数据存储的灵活性，Java 集合不仅可以用来存储不同类型不同数量的对象，还可以保存具有映射关系的数据
