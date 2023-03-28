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

#### String、StringBuffer、StringBuilder的区别

- **可变性**

`String` 类中使用 `final` 关键字修饰字符数组来保存字符串 `private final char value[]` ，所以 `String` 对象是不可变的

>在 Java 9 之后，String 、StringBuilder 与 StringBuffer 的实现改用 `byte` 数组存储字符串 private final byte[] value

而 `StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串 `char[]value` ，
但是没有用 `final` 关键字修饰，所以这两种对象都是可变的

- **线程安全性**

`String` 中的对象是不可变的，也就可以理解为常量，线程安全

`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的

`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的

- **性能**

每次对 `String` 类型进行改变的时候，都会生成一个新的 `String` 对象，然后将指针指向新的 `String` 对象

`StringBuffer` 每次都会对 `StringBuffer` 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 `StringBuilder` 相比使用 `StringBuffer` 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险


#### 为什么Java9后使用byte字节数组而舍弃了char字符数组

- todo

#### 等号与equals的区别

对于基本类型来说，== 比较的是值是否相等；

对于引用类型来说，== 比较的是两个引用是否指向同一个对象地址（两者在内存中存放的地址（堆内存地址）是否指向同一个地方）；


#### hashCode与equals的相关规定


- 如果两个对象相等，则 `hashcode` 一定也是相同的

- 两个对象相等，对 `equals()` 方法返回 true

- 两个对象有相同的 `hashcode` 值，它们也不一定是相等的

综上，`equals()` 方法被覆盖过，则 `hashCode()` 方法也必须被覆盖，
**hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写 `hashCode()`，则该 `class` 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）**

### 反射

#### 为什么说反射速度慢？为什么慢？

1. 反射会进行一系列的安全性校验，并且一些方法需要通过调用 native 方法来实现

2. 如果类没被加载，还得先加载类，并经过连接等阶段，而 new 则无需查找，因为在 Linking 阶段已经将符号引用转为直接引用

3. 反射调用方法时会从方法数组中遍历查找，并且会检查可见性等操作会耗时。

4. 反射在达到一定次数时（15），会动态编写字节码并加载到内存中，这个字节码没有经过编译器优化，也不能享受JIT优化。



### 注解

#### 说说你对 Java 注解的理解

注解是通过 `@interface` 关键字来进行定义的，形式和接口差不多，只是前面多了一个@

```java
public @interface TestAnnotation {
}
```

要使注解能正常工作，还需要使用元注解，它是可以用到注解上的注解。元标签有：

- **@Retention：** 说明注解的存活时间，取值有 
  - RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时被丢弃；
  - RetentionPolicy.CLASS  注解只保留到编译进行的时候，并不会被加载到 `JVM` 中
  - RetentionPolicy.RUNTIME 可以留到程序运行的时候，它会被加载进入到 `JVM` 中，所以在程序运行时可以获取到它们
- **@Documented：** 注解中的元素包含到 `javadoc` 中去
- **@Target：** 限定注解的应用场景
  - ElementType.FIELD 给属性进行注解；
  - ElementType.LOCAL_VARIABLE 可以给局部变量进行注解；
  - ElementType.METHOD 可以给方法进行注解；
  - ElementType.PACKAGE 可以给一个包进行注解 
  - ElementType.TYPE 可以给一个类型进行注解，如类、接口、枚举

- **@Inherited：** 若一个超类被 `@Inherited` 注解过的注解进行注解，它的子类没有被任何注解应用的话，该子类就可继承超类的注解；
- **@Repeatable：** 注解是 `Java 8` 新增加的，它允许在相同的程序元素中重复注解


#### 注解的作用

- 提供信息给编译器：编译器可利用注解来探测错误和警告信息
- 编译阶段：软件工具可以利用注解信息来生成代码、html 文档或做其它相应处理；
- 运行阶段：程序运行时可利用注解提取代码，注解是通过反射获取的，可以通过 `Class` 对象的 **isAnnotationPresent()** 方法判断它是否应用了
某个注解，再通过 **getAnnotation()** 方法获取 `Annotation` 对象


### 反射

#### 为什么说反射速度慢？为什么慢？

1. 反射会进行一系列的安全性校验，并且一些方法需要通过调用 native 方法来实现

2. 如果类没被加载，还得先加载类，并经过连接等阶段，而 new 则无需查找，因为在 Linking 阶段已经将符号引用转为直接引用

3. 反射调用方法时会从方法数组中遍历查找，并且会检查可见性等操作会耗时。

4. 反射在达到一定次数时（15），会动态编写字节码并加载到内存中，这个字节码没有经过编译器优化，也不能享受JIT优化。


### 注解
