
#### 比较HashSet、LinkedHashSet和TreeSet三者的异同

- `HashSet` 是 `Set` 接口的主要实现类 ，`HashSet` 的底层是 `HashMap`，线程不安全的，可以存储 `null` 值；

- `LinkedHashSet` 是 `HashSet` 的子类，能够按照添加的顺序遍历；

- `TreeSet` 底层使用红黑树，能够按照元素的顺序进行遍历，排序的方式有自然排序和定制排序。


#### HashSet是怎么去重的？

`HashSet` 底层是用 `HashMap` 来维护的，并将元素作为key，值则为一个固定的对象。所以当key值相同时，将不会被加入，这就是 `Set` 为社么能去重的原因。

当你把对象加入HashSet时，`HashSet` 会先计算对象的 `hashcode` 值来判断对象加入的位置，同时也会与其他加入的对象的 `hashcode` 值作比较，如果没有相符的 `hashcode`，
`HashSet` 会假设对象没有重复出现。但是如果发现有相同 `hashcode` 值的对象，这时会判断内存地址或调用 `equals()` 方法来检查对象相等的对象是否真的相同。
如果两者相同，`HashSet` 就不会让加入操作成功