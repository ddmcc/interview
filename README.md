
## Java基础

### String类和常量池

[String不可变性](http://ddmcc.cn/2021/02/14/string/)

[String字符串长度限制](http://ddmcc.cn/2021/02/27/length-limit-on-string/)

#### String对象的两种创建方式

```java
// 先检查字符串常量池中有没有"abcd"，如果字符串常量池中没有，则创建一个，然后 str1 指向字符串常量池中的对象，如果有，则直接将 str1 指向"abcd""；
String str1 = "abcd";
// 堆中创建一个新的对象
String str2 = new String("abcd");
// 堆中创建一个新的对象
String str3 = new String("abcd");
System.out.println(str1==str2);//false
System.out.println(str2==str3);//false
```

![markdown](https://ddmcc-1255635056.file.myqcloud.com/166201f2-b832-4e59-a7d0-5aa6f0659d8b.png)


- 第一种方式是在常量池中拿对象；
- 第二种方式是直接在堆内存空间创建一个新的对象。


**String 类型的常量池比较特殊。它的主要使用方法有两种：**

- 直接使用双引号声明出来的 String 对象会直接存储在常量池中。
- 如果不是用双引号声明的 String 对象，可以使用 String 提供的 intern() 方法。String.intern() 是一个 Native 方法，它的作用是：如果运行时常量池中已经包含一个等于此 String 对象内容的字符串，则返回常量池中该字符串的引用；如果没有，JDK1.7 之前（不包含 1.7）的处理方式是在常量池中创建与此 String 内容相同的字符串，并返回常量池中创建的字符串的引用，JDK1.7 以及之后的处理方式是在常量池中记录此字符串的引用，并返回该引用

```java
String str1 = "str";
String str2 = "ing";

String str3 = "str" + "ing";//常量池中的对象
String str4 = str1 + str2; //在堆上创建的新的对象
String str5 = "string";//常量池中的对象
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false

```

![markdown](https://ddmcc-1255635056.file.myqcloud.com/46f2e99a-54c0-47bc-9d0a-0fa627fe1a23.png)

尽量避免多个字符串拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 StringBuilder 或者 StringBuffer

#### newString这句话创建了几个对象？

将创建 1 或 2 个字符串。如果池中已存在字符串常量“abc”，则只会在堆空间创建一个字符串常量“abc”。如果池中没有字符串常量“abc”，那么它将首先在池中创建，然后在堆空间中创建，因此将创建总共 2 个字符串对象

**验证：**

```java
String s1 = new String("abc");// 堆内存的地址值
String s2 = "abc";
System.out.println(s1 == s2);// 输出 false,因为一个是堆内存，一个是常量池的内存，故两者是不同的。
System.out.println(s1.equals(s2));// 输出 true
```

**结果：**

```java
false
true
```

#### 为什么说反射速度慢？为什么慢？

1. 反射会进行一系列的安全性校验，并且一些方法需要通过调用 native 方法来实现

2. 如果类没被加载，还得先加载类，并经过连接等阶段，而 new 则无需查找，因为在 Linking 阶段已经将符号引用转为直接引用

3. 反射调用方法时会从方法数组中遍历查找，并且会检查可见性等操作会耗时。

4. 反射在达到一定次数时，会动态编写字节码并加载到内存中，这个字节码没有经过编译器优化，也不能享受JIT优化。


## 设计模式

#### 说说设计模式6大原则？

- **单一职责原则**

`核心思想`：应该有且仅有一个原因引起类的变更

`问题描述`：假如有类Class1完成职责T1，T2，当职责T1或T2有变更需要修改时，有可能影响到该类的另外一个职责正常工作

`好处`：类的复杂度降低、可读性提高、可维护性提高、扩展性提高、降低了变更引起的风险。


- **开闭原则**

>定义：类、模块、函数等应该是可以拓展的，但是不可修改。或者说修改也不能影响到现有的功能

`核心思想`：需要变化时，应该尽量通过拓展的方式来实现变化，而不是通过修改已有代码来实现


- **里氏替换原则**

>定义：所有引用基类的地方必须能透明地使用其子类的对象


`核心思想`：在使用基类的的地方可以任意使用其子类，能保证子类完美替换基类。

`通俗来讲`：只要父类能出现的地方子类就能出现。反之，父类则未必能胜任

采用里氏替换原则的好处可以让继承的 `好处` 发挥最大的作用（如：减少创建类的工作量，提供代码重用等），
并减少继承的 `“弊”` 带来的诸多麻烦（如：继承增强了耦合性，当需要对父类的代码进行修改时，必须考虑到对子类产生的影响）。这就要求：

- 子类必须实现父类的抽象方法，但不得重写（覆盖）父类的非抽象（已实现）方法

- 子类中可以增加自己特有的方法

- 当子类覆盖或实现父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松

- 当子类的方法实现父类的（抽象）方法时，方法的后置条件（即方法的返回值）要比父类更严格


**简单的来说就是尽量不修改父类继承而来的方法，如果要修改也要不影响父类和其它类**


- **依赖倒置原则**

`核心思想`：高层模块不应该依赖底层模块，二者都该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象

`说明`：高层模块就是调用端，低层模块就是具体实现类。抽象就是指接口或抽象类。细节就是实现类

`通俗来讲`：依赖倒置原则的本质就是通过抽象（接口或抽象类）使个各类或模块的实现彼此独立，互不影响，实现模块间的松耦合

`问题描述`：类A直接依赖类B，假如要将类A改为依赖类C，则必须通过修改类A的代码来达成。这种场景下，类A一般是高层模块，负责复杂的业务逻辑；类B和类C是低层模块，负责基本的原子操作；假如修改类A，会给程序带来不必要的风险。

`解决方案`：将类A修改为依赖接口interface，类B和类C各自实现接口interface，类A通过接口interface间接与类B或者类C发生联系，则会大大降低修改类A的几率


- **接口隔离原则**

`核心思想`：类间的依赖关系应该建立在最小的接口上

`通俗来讲`：建立单一接口，不要建立庞大臃肿的接口，尽量细化接口，接口中的方法尽量少。也就是说，我们要为各个类建立专用的接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用

    接口尽量小，但是要有限度 对接口进行细化可以提高程序设计灵活性，但是如果过小，则会造成接口数量过多，使设计复杂化。所以一定要适度

    提高内聚，减少对外交互 使接口用最少的方法去完成最多的事情

    为依赖接口的类定制服务 只暴露给调用的类它需要的方法，它不需要的方法则隐藏起来。只有专注地为一个模块提供定制服务，才能建立最小的依赖关系


- **迪米特原则**

`核心思想`：类间解耦

`通俗来讲`： 一个类应该对自己需要耦合或者调用的类知道越少越好。软件编程的总的原则：高内聚，低耦合

    一个处于松耦合中的类一旦被修改，则不会对关联的类造成太大的波及。

    在类的机构设计上， 每一个类都应当尽量降低其成员变量和成员函数的访问权限

    在对其他类的引用上， 一个类对其他对象的引用应当降到最低


**一句话总结：**

`单一职责原则` 告诉我们实现类要职责单一；`里氏替换原则`告诉我们不要破坏继承体系；`依赖倒置原则` 告诉我们要面向接口编程；`接口隔离原则` 告诉我们在设计接口的时候要精简单一；`迪米特法则` 告诉我们要降低耦合。而`开闭原则是总纲`，他告诉我们要对扩展开放，对修改关闭



#### 代理模式和桥接模式的异同点？


`代理`、`桥接`、`装饰器`、`适配器`，这 4 种模式是比较常用的 **结构型设计模式**。它们的代码结构非常相似。笼统来说，它们都可以称为 `Wrapper` 模式，也就是通过 `Wrapper` 类二次封装原始类。


`代理模式`：代理模式在不改变原始类接口的条件下，为原始类定义一个代理类，通过 `组合的方式` 来控制访问和增强

`桥接模式`：桥接模式的目的是将接口部分和实现部分分离，从而让它们可以较为容易、也相对独立地加以改变

`装饰器模式`：装饰者模式在不改变原始类接口的情况下，`通过继承的方式`，对原始类功能进行增强，并且支持多个装饰器的嵌套使用

`适配器模式`：适配器模式是一种事后的补救策略。适配器提供跟原始类不同的接口，而代理模式、装饰器模式提供的都是跟原始类相同的接口



#### 有几种单例的实现方式？以及它们的优缺点？

1）**饿汉式单例模式**

是最简单实现单例的方式，~~在类加载的时候就已经实现单例模式对象的生成，如果出现该单例对象占用内存很大但是从来未被调用的情况，该方式就会造成资源的浪费~~


2）**懒汉单例模式（线程不安全）**

它的优点是在第一次访问的时候才会被创建，避免了创建了没有使用而浪费资源的问题。但是每次访问都需要判断是否创建，也会影响性能。而且在 `多线程` 的情况下，它 并不是线程安全的。有可能会产生不同的对象


3）**懒汉同步式**

确保了每次只有一个线程进入方法，解决了线程安全的问题。但因为是同步方法，所以会很影响性能。而且我们只需要确保第一次访问的时候不被重复创建实例， 在第一次创建之后，同步方法就成了累赘了


4）**双重检查加锁**

这种方式是对前一种实现方式的改进，既保证了线程安全，也避免了性能影响，但是需要注意的是单例对象的引用需要用 `volatile` 关键字修饰，防止指令重排序，避免产生未初始化完全的对象


5）**静态内部类**

`优点`： 外部类加载时并不需要立即加载内部类，内部类不被加载则不去初始化instance，故而不占内存。即当SingletonPattenTest第一次被加载时， 并不需要去加载SingleTonHoler，只有当getInstance()方法第一次被调用时，才会加载SingleTonHoler类并初始化instance，这种方法不仅能确保线程安全，也能保证单例的唯一性，同时也延迟了单例的实例化。

`缺点`：

- 需要两个类去做到这一点，虽然不会创建静态内部类的对象，但是其 Class 对象还是会被创建，而且是属于方法区的对象

- 创建的单例，一旦在后期被销毁，不能重新创建


6）**注册表式**

用一个map来作为存储实例的载体，本质还是双重检查加锁的方式


7）**枚举式**

- 避免反射攻击的问题

- 避免线程安全问题。枚举类所有属性都会被声明称static类型，它是在类加载的时候初始化的，而类的加载和初始化过程都是线程安全的

- 避免序列化问题， 任何一个readObject方法，不管是显式的还是默认的，它都会返回一个新建的实例，这个新建的实例不同于该类初始化时创建的实例。枚举序列化的时候仅仅是将枚举对象的name属性输出到结果中，反序列化的时候则是通过java.lang.Enum的valueOf方法来根据名字查找枚举对象


## 集合容器

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


#### System.arraycopy()和Arrays.copyOf()方法

**`System.arraycopy()` 方法**

`arraycopy()` 方法是一个 `native` 方法，作用是从 **原数组中指定起始下标，复制 n 个元素到指定起始下标目标数组中**。

在 `ArrayList` 中主要用于移动数组元素和添加元素，如下图：arraycopy() 方法实现数组自己复制自己，将目标 `index` 后的元素往后移动一位，并将新的元素放到目标 `index`

![markdown](https://ddmcc-1255635056.file.myqcloud.com/d0d24a11-9423-48a7-92d9-02fce7db3fb2.png)


#### 比较HashSet、LinkedHashSet和TreeSet三者的异同

- `HashSet` 是 `Set` 接口的主要实现类 ，`HashSet` 的底层是 `HashMap`，线程不安全的，可以存储 `null` 值；

- `LinkedHashSet` 是 `HashSet` 的子类，能够按照添加的顺序遍历；

- `TreeSet` 底层使用红黑树，能够按照元素的顺序进行遍历，排序的方式有自然排序和定制排序。

>


## JVM

[Java虚拟机类加载过程](http://ddmcc.cn/2021/05/29/jvm-class-file-loading-process/)

### 类加载

#### 类的生命周期

一个类的生命周期包括加载、连接、初始化、使用、卸载5个阶段，其中连接阶段包括验证、准备、解析，具体如下：

![markdown](https://ddmcc-1255635056.file.myqcloud.com/2784440a-9233-4742-bf09-2da0d6e56250.png)


#### [类加载过程](http://ddmcc.cn/2021/05/29/jvm-class-file-loading-process/)

类加载过程包括：`加载（Loading）`、`连接（Linking）`、`初始化（Initialization）`3个阶段。其中连接过程又可以分为
验证（Verification）、准备（Preparation）、解析（Resolution）三个阶段

**加载阶段** 主要是通过类的全限定名来查找这个类的文件，将静态的类文件存储结构转化为运行时数据结构，并在内存中生成一个代表这个类的 java.lang.Class 对象，作为这个类的数据访问入口

这一步我们可以自定义类加载器去控制类文件的获取方式（重写一个类加载器的 loadClass() 方法）。数组类型不通过类加载器创建，它由 Java 虚拟机直接创建

**验证阶段** 的目的主要为了确保文件中的表示满足 静态语法或结构的约束 ，及安全性校验。主要包括文件格式验证、元数据验证、字节码验证、符号引用验证

**准备阶段** 主要为类或接口的静态字段分配内存，并将此类字段初始化为其默认值，**变量所使用的内存都将在 方法区 进行分配**。静态字段的显示初始化会在初始化阶段（Initialization）进行，而不是在准备阶段。并且准备阶段可以在加载后的任何时候进行，但必须在初始化之前完成

**解析阶段** 是虚拟机将常量池内的符号引用替换为直接引用的过程，也就是得到类或者字段、方法在内存中的指针或者偏移量

**初始化阶段** 就是执行初始化方法 `<clinit> ()` 方法的过程，是类加载的最后一步

#### 双亲委派加载机制

每一个类都有一个对应它的类加载器。系统中的 `ClassLoder` 在协同工作的时候会默认使用 `双亲委派模型` 。即在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。
加载的时候，首先会把该请求委派该父类加载器的 `loadClass()` 处理，因此所有的请求最终都应该传送到顶层的启动类加载器 `BootstrapClassLoader` 中。当父类加载器无法处理时，才由自己的 `findClass()` 方法来处理。当父类加载器为null时，会使用启动类加载器 `BootstrapClassLoader` 作为父类加载器

#### 如何实现一个自定义类加载器？

继承 `ClassLoader` 类，然后重写 `findClass` 方法

#### 如果不想用双亲委派模型怎么办？

自定义加载器的话，需要继承 `ClassLoader` 。如果我们不想打破双亲委派模型，就重写 `ClassLoader` 类中的 `findClass()` 方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。
但是，如果想打破双亲委派模型则需要重写 loadClass() 方法

#### 双亲委派模型的好处？

可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），
也保证了 `Java` 的核心 API 不被篡改。如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现多个不同的 `Object` 类


### JVM内存区域

#### 运行时数据区

**1.8之前：**

![markdown](https://ddmcc-1255635056.file.myqcloud.com/0ae211fc-8a32-4e8c-affa-9202418c4c05.png)


**1.8：**

![markdown](https://ddmcc-1255635056.file.myqcloud.com/06512b0c-a37c-4470-8940-05b256e1d1e7.png)


- **程序计数器：**

程序计数器只要有两个作用：

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了

>程序计数器是唯一一个不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡


- **虚拟机栈：**

Java 虚拟机栈也是线程私有的，它的生命周期和线程相同。实际上，虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接（调用本地方法时，链接到本地方法栈的方法）、方法出口信息。

局部变量表主要存放了编译期可知的各种数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）


>**Java 虚拟机栈会出现两种错误：StackOverFlowError 和 OutOfMemoryError：**
>
>`StackOverFlowError`： 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误
>
>`OutOfMemoryError`： Java 虚拟机栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常异常


- **本地方法栈：**

和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务**

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。
方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 `StackOverFlowError` 和 `OutOfMemoryError` 两种错误


- **堆：**

Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存**

>随着 JIT 编译期的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。
>从 JDK 1.7 开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存

Java 堆是垃圾收集器管理的主要区域，因此也被称作GC 堆（Garbage Collected Heap）。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，
所以 Java 堆还可以细分为：新生代和老年代：再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存**

在 JDK 7 版本及 JDK 7 版本之前，堆内存被通常被分为新生代、老年代和永久代。JDK 8 版本之后方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存

堆这里最容易出现的就是 OutOfMemoryError 错误，并且出现这种错误之后的表现形式还会有几种，比如：

>`OutOfMemoryError`: `GC Overhead Limit Exceeded` ： 当 JVM 花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误
>
>`java.lang.OutOfMemoryError`: `Java heap space` :假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发java.lang.OutOfMemoryError: Java heap space 错误

- **方法区：**

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据


- **运行时常量池：**

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池表（用于存放编译期生成的各种字面量和符号引用）

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 错误

>1. JDK1.7 之前运行时常量池逻辑包含字符串常量池存放在方法区, 此时 hotspot 虚拟机对方法区的实现为永久代
>2. JDK1.7 字符串常量池被从方法区拿到了堆中, **这里没有提到运行时常量池，也就是说字符串常量池被单独拿到堆，运行时常量池剩下的东西还在方法区**， 也就是 hotspot 中的永久代
>3. JDK1.8 hotspot 移除了永久代用元空间(Metaspace)取而代之，这时候字符串常量池还在堆，运行时常量池还在方法区,，只不过方法区的实现从永久代变成了元空间(Metaspace)


#### 深拷贝与浅拷贝

- 浅拷贝

**会为被复制的对象新开辟内存，但其所有属性的值会与原对象相同**

即如果属性是基本类型，拷贝的就是基本类型的值；如果属性是引用类型（堆内存地址），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象
>对于外层对象来说是深拷贝，对象里的引用类型变量来说是浅拷贝

- 深拷贝

深拷贝会拷贝所有的属性，如果属性是引用类型，则会为其新申请一块内存，并将内存地址赋值给属性


**Object的clone方法是深拷贝还是浅拷贝？如果要实现深拷贝怎么实现？**

Object的clone方法是浅拷贝，如果要实现深拷贝可以实现 **Cloneable** 接口，并重写 `Object.clone` 方法，自己实现深克隆。或者使用序列化/反序列化的方式进行深拷贝


**ArrayList集合、HashMap、Arrays.copyOf()方法等是深拷贝还是浅拷贝？** 

对于 `ArrayList` 来说，内部是用数组实现的，ArrayList对象本身和内部的数组对象都是新的对象，但其中的元素还是引用同一个地址，所以是浅拷贝

`HashMap` 也只是浅拷贝，键和值本身都不是克隆的

`Arrays.copyOf()` 返回的数组对象是新的对象，里面的元素也还是引用同一个地址，所以也是浅拷贝


#### 堆和栈的区别？

- `物理地址`

堆的物理地址分配对对象是不连续的。在GC的时候也要考虑到不连续的分配，所以有各种算法。比如，标记-消除，复制，标记-压缩，分代（即新生代使用复制算法，老年代使用标记——压缩）

栈使用的是数据结构中的栈，先进后出的原则，物理地址分配是连续的。所以性能快。

- `内存分别`

堆因为是不连续的，所以分配的内存是在运行期确认的，因此大小不固定。一般堆大小远远大于栈

栈是连续的，所以分配的内存大小要在编译期就确认，大小是固定的

- `存放的内容`

堆存放的是对象的实例和数组。因此该区更关注的是数据的存储

栈存放：局部变量，操作数栈，返回结果。该区更关注的是程序方法的执行

- `程序的可见度`

堆对于整个应用程序都是共享、可见的。

栈只对于线程是可见的。所以也是线程私有。他的生命周期和线程相同

- `异常`

当堆栈内存满时，Java运行时抛出 `Java.lang.StackOverFlowerError`，而如果堆内存满，则抛出 `Java.lang.OutOfMemoryError`:Java堆空间错误。



#### 方法是如何调用的？

Java 栈可用类比数据结构中栈，Java 栈中保存的主要内容是 `栈帧`，每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后，都会有一个栈帧被弹出。
栈帧中保存着一些执行方法的数据，如：局部变量表、操作数栈、动态链接、返回地址等

Java 方法有两种返回方式：

- return 语句
- 抛出异常

不管哪种返回方式都会导致栈帧被弹出


#### 方法区和永久代的关系

方法区是 Java 虚拟机规范中的定义，是一种规范，永久代是 HotSpot 的概念，是 HotSpot 对方法区的实现。，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法


#### 为什么要将永久代替换为元空间呢？

1. 整个永久代有一个 JVM 本身设置的固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小

>当元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace` 
>
>`-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制

2. 元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 MaxPermSize 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了


### 内存分配与垃圾回收

#### 对象内存分配机制

![markdown](https://ddmcc-1255635056.file.myqcloud.com/95d86b44-95d5-453c-917d-61106f3a5a67.png)

- `对象优先在 eden 区分配`

目前主流的垃圾收集器都会采用分代回收算法，因此需要将堆内存分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。针对新生代存活时间短对象

**大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1（分配到当时的FROM 区），
并且对象的年龄还会加 1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄达到15或达到动态晋升年龄阈值，就会被晋升到老年代中**


>"From"和"To"会交换他们的角色，也就是新的"To"就是上次 GC 前的“From”，新的"From"就是上次 GC 前的"To"。不管怎样，都会保证名为 To 的 Survivor 区域是空的。Minor GC 会一直重复这样的过程，直到“To”区被填满，"To"区被填满之后，会将所有对象移动到老年代中

- `大对象直接进入老年代`

大对象就是需要大量连续内存空间的对象（比如：字符串、数组）

**为什么要这样呢？**

为了避免为大对象分配内存时由于分配担保机制带来的复制而降低效率


- `长期存活的对象将进入老年代`

既然采用分代回收算法，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了做到这一点，虚拟机给每个对象一个对象年龄（Age）计数器

对象在 Survivor 中每熬过一次 MinorGC，年龄就增加 1 岁，当它的年龄增加到一定程度（默认为 15 可以通过参数 -XX:MaxTenuringThreshold 来设置 ）或者达到 `动态晋升年龄阀值`，就会被晋升到老年代中。


#### 动态计算晋升年龄阈值

遍历所有 `Survivor` 区域对象，按照 **年龄段从小到大对其所占用的大小进行累积** ，当加入某个年龄段后，累加的占用大小超过默认的 `-XX:TargetSurvivorRatio`（50%）时，取这个年龄和 `-XX:MaxTenuringThreshold` (15)中更小的一个值，作为新的晋升年龄阈值

```java
uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
  // survivor_capacity是survivor空间的内存大小
  // TargetSurvivorRatio survivor空间存活大小比例
  // desired_survivor_size 根据比例计算出期望存活对象内存总大小
  size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
  size_t total = 0;
  uint age = 1;
  while (age < table_size) {
    // sizes数组是每个年龄段对象大小，根据年龄取出整个年龄段所占用的内存大小
    total += sizes[age];
    // 直到累计总内存大于期望的占用大小
    if (total > desired_survivor_size) break;
    age++;
  }
  // 用当前age和MaxTenuringThreshold 对比找出最小值作为结果
  uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
    ...
}
```


#### 垃圾回收算法

**标记-清除算法**

标记-清除分为“标记”和“清除”阶段：**首先标记出所有不需要回收的对象**，在标记完成后统一回收掉所有没有被标记的对象。它是最基础的收集算法，后续的算法都是对其不足进行改进得到。

这种垃圾收集算法会带来两个明显的问题：

- `效率问题`
- `空间问题（标记清除后会产生大量不连续的碎片）`


**标记-复制算法**

为了解决效率问题，“标记-复制”收集算法出现了。它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收


**标记-整理算法**

根据老年代的特点提出的一种标记算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内


**分代收集算法**

当前虚拟机的垃圾收集都采用分代收集算法，只是根据对象存活周期的不同将内存分为几块。一般将 java 堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

**比如在新生代中，每次收集都会有大量对象死去，所以可以选择”标记-复制“算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清除”或“标记-整理”算法进行垃圾收集**



#### 为什么堆内存要分为新生代、老年代？

其实不分代完全可以，分代的好处就是优化GC性能。如果没有分代，所有的对象都在一块，这样就会对堆的所有区域进行扫描。而我们的很多对象都是朝生夕死的，如果分代的话，我们把新创建的对象放到某一地方，当GC的时候先把这块存“朝生夕死”对象的区域进行回收，这样就会腾出很大的空间出来

**所以这样进一步划分的目的是更好地回收内存，或者更快地分配内存**


#### 为什么要有Survivor区？

预筛选避免每进行一次 `Minor GC`，存活的对象就被送到老年代

Survivor的存在意义，就是减少被送到老年代的对象，进而减少Full GC的发生，`Survivor` 的预筛选保证，只有经历一定次数 `Minor GC` 还能在新生代中存活的对象，才会被送到老年代

但是你可能会说不放`Survivor`，每次gc再判断对象的年龄行不行？这样的话每次都要扫描整个新生代，gc消耗的时间变长

**为什么要设置两个Survivor区**

设置两个Survivor区最大的好处就是解决了碎片化。新生代一般采用 `标记-复制` 的回收算法。但是复制算法必须要有另一块内存作为存放对象，来避免碎片化的发生


#### 新生代gc工作流程

![markdown](https://ddmcc-1255635056.file.myqcloud.com/9e289f4e-7e61-49bc-97a6-9d5ac4d0be6f.png)


HotSpot JVM把年轻代分为了三部分：`1个Eden区`和`2个Survivor区`（分别叫from和to）。默认比例为8（Eden）：1（一个survivor）

一般情况下，新创建的对象都会被分配到Eden区(一些大对象特殊处理)，这些对象经过第一次Minor GC后，如果仍然存活，将会被移到Survivor区。对象在Survivor区中每熬过一次Minor GC，年龄就会增加1岁，当它的年龄增加到一定程度时，就会被移动到年老代中。

因为年轻代中的对象基本都是朝生夕死的(80%以上)，所以在年轻代的垃圾回收算法使用的是复制算法，复制算法的基本思想就是将内存分为两块，每次只用其中一块，当这一块内存用完，就将还活着的对象复制到另外一块上面。复制算法不会产生内存碎片

**在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中**


#### 如何判断对象是否死亡（两种方法）

- 引用计数法：

给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加 1；当引用失效，计数器就减 1；任何时候计数器为 0 的对象就是不可能再被使用的
    这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题
- 可达性分析算法：

基本思想就是通过一系列的称为 “GC Roots” 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的


>可作为 GC Roots 的对象包括下面几种:
>
>虚拟机栈(栈帧中的本地变量表)中引用的对象
>
>本地方法栈(Native 方法)中引用的对象
>
>方法区中类静态属性引用的对象
>
>方法区中常量引用的对象
>
>所有被同步锁持有的对象



#### MinorGc和FullGC有什么不同呢？

Partial GC：并不收集整个GC堆的模式

- Young GC：只收集young gen的GC
- Old GC：只收集old gen的GC。只有CMS的concurrent collection是这个模式
- Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式

Full GC：收集整个堆，包括young gen、old gen、perm gen（如果存在的话）等所有部分的模式。


#### 什么是空间分配担保机制？

在发生 `Minor GC` 之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那么 `Minor GC` 可以确保是安全的。
如果不成立，则虚拟机会查看 `HandlePromotionFailure` 设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次 `Minor GC`，尽管这次 `Minor GC` 是有风险的；如果小于，或者 `HandlePromotionFailure`设置不允许冒险，那这时也要改为进行一次 `Full GC` 。

>上面说的风险是什么呢？我们知道，新生代使用复制收集算法，但为了内存利用率，只使用其中一个Survivor空间来作为轮换备份，因此当出现大量对象在Minor GC后仍然存活的情况（最极端的情况就是内存回收后新生代中所有对象都存活），就需要老年代进行分配担保，把Survivor无法容纳的对象直接进入老年代



### Java对象


#### 对象的创建过程

![markdown](https://ddmcc-1255635056.file.myqcloud.com/629b9d68-355f-400b-8717-b3b47a12a0ae.png)

**Step1:类加载检查**

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程


**Step2:分配内存**

在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来


**Step3:初始化零值**

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值


**Step4:设置对象头**

初始化零值完成之后，虚拟机要对对象进行必要的设置，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄、锁等信息。 这些信息存放在对象头中。 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式


**Step5:执行 init 方法**

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，<init> 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 <init> 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来



#### 对象内存的分配方式？

分配方式有 `指针碰撞` 和 `空闲列表` 两种

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的

![markdown](https://ddmcc-1255635056.file.myqcloud.com/a209c95b-510b-432f-aabf-e2740d9dd7ee.png)


**内存分配并发问题**

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

- **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。
- **TLAB：** 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配


#### 对象的内存布局

在 Hotspot 虚拟机中，对象在内存中的布局可以分为 3 块区域：`对象头`、`实例数据`和`对齐填充`

**对象头** 包括两部分信息：

- 第一部分(mark word 8字节)用于存储对象自身的运行时数据（哈希码、GC 分代年龄、锁状态标志等等）

- 另一部分是类型指针（class pointer 8字节），即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例

- length：**数组对象持有** 大小占用4字节

**实例数据** 部分是对象真正存储的有效信息，也是在程序中所定义的各种类型的字段内容，包含所有成员变量

>boolean和byte：1字节
>
>short和char：2字节
>
>int和float：4字节
>
>long和double：8字节 
>
>reference：8字节

**对齐填充部分** 仅仅起占位作用。 因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，
换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全


#### 对象的访问定位

对象的访问方式由虚拟机实现而定，目前主流的访问方式有 `使用句柄` 和 `直接指针` 两种：

1. **句柄：** 如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息

![markdown](https://ddmcc-1255635056.file.myqcloud.com/6cc845e2-3e56-4108-88f1-4b956ea68217.png)

2. **直接指针：** 如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而 reference 中存储的直接就是对象的地址

![markdown](https://ddmcc-1255635056.file.myqcloud.com/03175ed3-cf2a-4aa6-81f7-637cbdf6ec33.png)


**这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。
使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销**


#### Java中有哪些引用类型？

`强引用：` 发生 gc 的时候不会被回收

`软引用：` 有用但不是必须的对象，在发生内存溢出之前会被回收

`弱引用：` 有用但不是必须的对象，在下一次GC时会被回收

`虚引用（幽灵引用/幻影引用）：` 无法通过虚引用获得对象，用 PhantomReference 实现虚引用，虚引用的用途是在 gc 时返回一个通知


### 垃圾回收器

- **`串行（Serial）回收器是单线程的一个回收器，简单、易实现、效率高`**

![markdown](https://ddmcc-1255635056.file.myqcloud.com/2c00d3c5-88f7-444c-802a-8d44eb049453.png)

它的 “单线程” 的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（ "Stop The World" ），直到它收集结束

**新生代采用标记-复制算法，老年代采用标记-整理算法**

- **`并行（ParNew）回收器是Serial的多线程版，可以充分的利用CPU资源，减少回收的时间`**

![markdown](https://ddmcc-1255635056.file.myqcloud.com/914abf09-dd8d-4e59-8637-189025c5e73b.png)

除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和 Serial 收集器完全一样

**新生代采用标记-复制算法，老年代采用标记-整理算法**

- **`吞吐量优先（Parallel Scavenge）回收器，侧重于吞吐量的控制，JDK 1.8 默认使用的是 Parallel Scavenge + Parallel Old`**

![markdown](https://ddmcc-1255635056.file.myqcloud.com/31ad7c3f-a04b-4f58-8fec-6a8613337779.png)


**新生代采用标记-复制算法，老年代采用标记-整理算法**

- **`并发标记清除（CMS，Concurrent Mark Sweep）回收器是一种以获取最短回收停顿时间为目标的回收器，该回收器是基于“标记-清除”算法实现的`**

![markdown](https://ddmcc-1255635056.file.myqcloud.com/a2cd0a8d-aba1-4d98-9276-2f3acd59b69c.png)

它的运作过程相比于前面几种垃圾收集器来说更加复杂一些。整个过程分为四个步骤：

- 初始标记： 暂停所有的其他线程，并记录下直接与 root 相连的对象，速度很快 ；
- 并发标记： 同时开启 GC 和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以 GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
- 重新标记： 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
- 并发清除： 开启用户线程，同时 GC 线程开始对未标记的区域做清扫。


主要优点：**并发收集**、**低停顿**。但是它有下面三个明显的缺点：

- 对 CPU 资源敏感；
- 无法处理浮动垃圾；
- 它使用的回收算法-“标记-清除”算法会导致收集结束时会有大量空间碎片产生


- **`G1 收集器`**

被视为 JDK1.7 中 HotSpot 虚拟机的一个重要进化特征。它具备一下特点：

- **并行与并发**：G1 能充分利用 CPU、多核环境下的硬件优势，使用多个 CPU（CPU 或者 CPU 核心）来缩短 Stop-The-World 停顿时间。部分其他收集器原本需要停顿 Java 线程执行的 GC 动作，G1 收集器仍然可以通过并发的方式让 java 程序继续执行。
- **分代收集**：虽然 G1 可以不需要其他收集器配合就能独立管理整个 GC 堆，但是还是保留了分代的概念。
- **空间整合**：与 CMS 的“标记-清理”算法不同，G1 从整体来看是基于“标记-整理”算法实现的收集器；从局部上来看是基于“标记-复制”算法实现的。
- **可预测的停顿**：这是 G1 相对于 CMS 的另一个大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但 G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内。

G1 收集器的运作大致分为以下几个步骤：

- **初始标记**
- **并发标记**
- **最终标记**
- **筛选回收**

G1 收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的 Region(这也就是它的名字 Garbage-First 的由来) 。这种使用 Region 划分内存空间以及有优先级的区域回收方式，保证了 G1 收集器在有限时间内可以尽可能高的收集效率（把内存化整为零

## 网络协议

### TCP

#### TCP是什么？

`面向连接`，`可靠的传输协议`

#### TCP三次握手协议

- 客户端–发送带有 SYN 标志的数据包–一次握手–服务端
- 服务端–发送带有 SYN/ACK（syn + 1） 标志的数据包–二次握手–客户端
- 客户端–发送带有带有 ACK（syn + 1） 标志的数据包–三次握手–服务端


#### 为什么要三次握手？

三次握手的目的是建立可靠的通信信道，而三次握手最主要的目的就是双方确认自己与对方的发送与接收是正常的

第一次SYN+ACK保证客户端到服务端发送接收数据是正常的。服务端发送SYN是为了保证服务端到客户端发送数据也是正常的
 
 
#### 第2次握手传回了ACK，为什么还要传回SYN？

接收端传回发送端所发送的ACK是为了告诉客户端，我接收到的信息确实就是你所发送的信号了，这表明从客户端到服务端的通信是正常的。而回传SYN则是为了建立并确认从服务端到客户端的通信


#### 四次挥手协议

- 客户端-发送一个 FIN，用来关闭客户端到服务器的数据传送
- 服务器-收到这个 FIN，它发回一 个 ACK，确认序号为收到的序号加1 。和 SYN 一样，一个 FIN 将占用一个序号
- 服务器-关闭与客户端的连接，发送一个FIN给客户端
- 客户端-发回 ACK 报文确认，并将确认序号设置为收到序号加1


#### 为什么要四次挥手？

任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了TCP连接

举个例子：A 和 B 打电话，通话即将结束后，A 说“我没啥要说的了”，B回答“我知道了”，但是 B 可能还会有要说的话，A 不能要求 B 跟着自己的节奏结束通话，于是 B 可能又巴拉巴拉说了一通，最后 B 说“我说完了”，A 回答“知道了”，这样通话才算结束


#### HTTP长连接，短连接

在HTTP/1.0中默认使用短连接。也就是说，客户端和服务器每进行一次HTTP操作，就建立一次连接，任务结束就中断连接。当客户端浏览器访问的某个HTML或其他类型的Web页中包含有其他的Web资源（如JavaScript文件、图像文件、CSS文件等），每遇到这样一个Web资源，浏览器就会重新建立一个HTTP会话

而从HTTP/1.1起，默认使用长连接，用以保持连接特性，使用长连接的HTTP协议，会在响应头加入这行代码：`Connection:keep-alive`
在使用长连接的情况下，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，客户端再次访问这个服务器时，会继续使用这一条已经建立的连接。Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。实现长连接需要客户端和服务端都支持长连接

**HTTP协议的长连接和短连接，实质上是TCP协议的长连接和短连接**


#### HTTP1.0和HTTP1.1的主要区别是什么?

长连接 : 在HTTP/1.0中，默认使用的是短连接，HTTP/1.1的持续连接有非流水线方式和流水线方式流水线方式是客户在收到HTTP的响应报文之前就能接着发送新的请求报文。与之相对应的非流水线方式是客户在收到前一个响应后才能发送下一个请求

**错误状态响应码 :** 在HTTP1.1中新增了24个错误状态响应码

**带宽优化及网络连接的使用：** HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206

#### HTTP和HTTPS的区别？

**端口** ：HTTP的URL由“http://”起始且默认使用端口80，而HTTPS的URL由“https://”起始且默认使用端口443。

**安全性和资源消耗：** HTTP协议运行在TCP之上，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份。HTTPS是运行在SSL/TLS之上的HTTP协议，SSL/TLS 运行在TCP之上。所有传输的内容都经过加密，加密采用对称加密，但对称加密的密钥用服务器方的证书进行了非对称加密。所以说，HTTP 安全性没有 HTTPS高，但是 HTTPS 比HTTP耗费更多服务器资源


## 浏览器

#### Http无状态链接，如何实现session跟踪？

大部分情况下，我们都是通过在 Cookie 中附加一个 Session ID 来方式来跟踪

- Cookie 被禁用怎么办?
- 最常用的就是利用 URL 重写把 Session ID 直接附加在URL路径的后面

#### Cookie和Session的区别

- **存储位置：** Cookie 存储在客户端中，而Session存储在服务器上
- **安全性：** 相对来说 Session 安全性更高。如果要在 Cookie 中存储一些敏感信息，不要直接写入 Cookie 中，最好能将 Cookie 信息加密然后使用到的时候再去服务器端解
- **作用：** Cookie 一般用来保存用户信息比如 token，Session 的主要作用就是通过服务端记录用户的状态

#### 在浏览器中输入url地址->>显示主页的过程

1. DNS解析：获取域名对应IP
2. TCP连接：三次握手
3. 发送HTTP请求
4. 服务器处理请求并返回HTTP报文
5. 浏览器解析渲染页面
6. 连接结束


## 并发编程


### ThreadLocal类


#### 说说ThreadLocal类？作用？数据结构？

- `作用`

`ThreadLocal` 对象可以存储线程局部变量，每个线程 `Thread` 拥有一份自己的副本变量，多个线程互不干扰

- `数据结构`

`Thread` 类有一个类型为 `ThreadLocal.ThreadLocalMap` 的实例变量 `threadLocals` ，也就是说每个线程有一个自己的 `ThreadLocalMap。`

`ThreadLocalMap` 有自己的独立实现，可以简单地将它的 `key` 视作 `ThreadLocal`，`value` 为代码中放入的值（实际上key并不是ThreadLocal本身，而是它的一个弱引用）。

每个线程在往 `ThreadLocal` 里放值的时候，都会往自己的 `ThreadLocalMap` 里存，读也是以 `ThreadLocal` 作为 `key`，在自己的 `map` 里找对应的 `key`，从而实现了线程隔离。

`ThreadLocalMap` 有点类似 `HashMap` 的结构，只是 `HashMap` 是由数组+链表实现的，而 `ThreadLocalMap` 中并没有链表结构。
它内部是由一个 `Entry` 数组来实现的， 它的key是ThreadLocal<?> k ，继承自WeakReference， 也就是我们常说的弱引用类型，值则为我们存储的值


#### ThreadLocalMap哈希算法？怎么解决hash冲突？

**hash算法**

在 `ThreadLocal` 中有一个增量属性为 `HASH_INCREMENT = 0x61c88647`， 每当创建一个ThreadLocal对象，这个ThreadLocal.nextHashCode 这个值就会增长 `0x61c88647` 。
这个值很特殊，它是 **斐波那契数** 也叫 **黄金分割数**。`hash` 增量为 这个数字，带来的好处就是 `hash` 分布非常均匀。在数组中的索引通过 `hashcode & (length - 1)` 计算得到

**怎么解决hash冲突**
 
虽然 `ThreadLocalMap` 中使用了 **黄金分割数** 来作为 `hash` 计算因子，大大减少了 `Hash` 冲突的概率，但是仍然会存在冲突。
`HashMap` 中解决冲突的方法是在数组上构造一个链表结构，冲突的数据挂载到链表上，如果链表长度超过一定数量则会转化成红黑树。而 `ThreadLocalMap` 中并没有链表结构，所以这里不能适用 `HashMap` 解决冲突的方式

当往 `map` 中 `set` 时，会遍历当前的数组，直到 `Entry` 为空，有几种情况：

- 通过 `hash` 计算后的槽位对应的 `Entry` 数据为空：停止遍历，直接将数据放到该槽位

- 槽位数据不为空，`key` 值与当前计算获取的key值一致：**直接更新该槽位的数据**

- 槽位数据不为空，往后遍历过程中，在找到Entry为null的槽位之前，遇到 `key = null` 的Entry: 则开始替换过期数据逻辑，从 `key = null` 的槽位开始向后遍历，进行探测式数据清理工作


#### ThreadLocalMap扩容机制？

当容量超过 `threshold = len * 2 / 3` 即超过 `2/3` 时就开始执行 `rehash()`, 先进行探测式数据清理，然后在判断容量超过 `threshold - threshold / 4` 即当前容量超过一半时执行
`resize()`，**扩容后的大小为原来的两倍**


#### ThreadLocal内存泄露问题？源码是怎么解决的？为什么要用弱引用？

- `解决key内存泄露的问题`

ThreadLocal中，获取到线程私有变量是通过线程持有的一个threadLocalMap，然后传入ThreadLocal当做key获取到对象的，这时候就有个问题，如果你在使用完 ThreadLocal 之后，将其置为null，这时候这个对象并不能被回收，
因为他还有 ThreadLocalMap -> entry -> key的引用，直到该线程被销毁，但是这个线程很可能会被放到线程池中不会被销毁，这就产生了内存泄露。jdk是通过弱引用来解决的这个问题的，
entry中对key的引用是弱引用，当你取消了ThreadLocal的强引用之后，他就只剩下一个弱引用了，所以也会被回收。**所以弱引用主要用来解决key内存泄露的问题**

>弱引用在下一次GC时会被回收，不管内存是否足够


- `解决值内存泄露`


因为key是弱引用，在外部没有强引用的情况下，下一次GC时就会被回收，就会存在(null, value)的情况，又因为还有 ThreadLocalMap -> entry -> value的引用，直到该线程被销毁，但是这个线程很可能会被放到线程池中不会被销毁，这就产生了内存泄露
源码是通过在，`get`，`set`，`remove` 等方法时去检测删除过期的 `Entry` 即 null key 的


#### ThreadLocal使用场景？

**解决线程安全问题**


**传递线程上下文**
