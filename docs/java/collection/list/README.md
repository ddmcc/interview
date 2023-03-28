### List


#### ArrayList和Vector的区别

- `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[]` 存储，适用于频繁的查找工作，线程不安全 ；

- `Vector` 是 `List` 的古老实现类，底层使用 `Object[]` 存储，线程安全的。


#### ArrayList与LinkedList区别

- **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；

- **底层数据结构：**  `Arraylist` 底层使用的是 `Object` 数组；`LinkedList` 底层使用的是 双向链表 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！）

- **插入和删除是否受元素位置的影响：**

`ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行add(E e)方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（add(int index, E element)）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。

`LinkedList` 采用链表存储，所以，如果是在头尾插入或者删除元素不受元素位置的影响（add(E e)、addFirst(E e)、addLast(E e)、removeFirst() 、 removeLast()），近似 O(1)，如果是要在指定位置 i 插入和删除元素的话（add(int index, E element)，remove(Object o)） 时间复杂度近似为 O(n) ，因为需要先移动到指定位置再插入。
是否支持快速随机访问： `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持

>快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index)方法)。

- **内存空间占用**： `ArrayList` 的空间浪费主要体现在在 `list` 列表的结尾会预留一定的容量空间，而 `LinkedList` 的空间花费则体现在它的每一个元素都需要消耗比 `ArrayList` 更多的空间（因为要存放直接后继和直接前驱以及数据）


#### RandomAccess接口的作用是什么？

源码上 `RandomAccess` 接口中什么都没有定义。所以，在我看来 `RandomAccess` 接口不过是一个标识罢了。标识什么？ 标识实现这个接口的类具有 **随机访问功能。**
并不是说 `ArrayList` 实现 `RandomAccess` 接口才具有快速随机访问功能的，
`ArrayList` 底层是数组，而 `LinkedList` 底层是链表。数组天然支持随机访问，时间复杂度为 O(1)，所以称为 **快速随机访问**。链表需要遍历到特定位置才能访问特定位置的元素，时间复杂度为 O(n)，所以不支持快速随机访问

在 binarySearch() 方法中，它要判断传入的 list 是否 `RamdomAccess` 的实例，如果是，调用 `indexedBinarySearch()` 方法，如果不是，那么调用 `iteratorBinarySearch()` 方法


#### 说一说ArrayList的扩容机制吧

**以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为 10
直到添加第 11 个元素，minCapacity(为 11)比 elementData.length（为 10）要大。进入 grow 方法进行扩容**

`ArrayList` 每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity 为偶数就是 1.5 倍，否则是 1.5 倍左右）


```java
// minCapacity 最小容量 oldCapacity + 1
// oldCapacity >> 1  偶数为一半，奇数不到一半
int newCapacity = ArraysSupport.newLength(oldCapacity, minCapacity - oldCapacity, oldCapacity >> 1);

// minGrowth = minCapacity - oldCapacit = 1
// prefGrowth = oldCapacity >> 1
// oldLength = oldCapacity
int newLength = Math.max(minGrowth, prefGrowth) + oldLength;
```


#### 说说ensureCapacity方法的作用

`ensureCapacity(int minCapacity)` 方法在 `ArrayList` 类内部并没有被调用，所以它是提供给外部使用的。**作用：** 是扩容数组，以至少容纳 `minCapacity` 大小的元素，
**条件是：** `minCapacity` 大小大于当前数组个数并且当前数组已有元素或 `minCapacity` > `DEFAULT_CAPACITY`

**所以可以在大量增加元素之前调用 `ensureCapacity(int minCapacity)`，一次性扩容完毕，减少扩容次数。**


#### arraycopy和copyof方法

**`System.arraycopy()` 方法**

`arraycopy()` 方法是一个 `native` 方法，作用是从 **原数组中指定起始下标，复制 n 个元素到指定起始下标目标数组中**。

在 `ArrayList` 中主要用于移动数组元素和添加元素，如下图：arraycopy() 方法实现数组自己复制自己，将目标 `index` 后的元素往后移动一位，并将新的元素放到目标 `index`

![markdown](https://ddmcc-1255635056.file.myqcloud.com/d0d24a11-9423-48a7-92d9-02fce7db3fb2.png)

