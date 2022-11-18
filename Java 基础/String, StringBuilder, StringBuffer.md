参考 https://cloud.tencent.com/developer/article/1645883

# 1、String

String 表示的就是 Java 中的字符串，我们日常开发用到的使用 `""` 双引号包围的数都是字符串的实例。String 类其实是通过 char 数组来保存字符串的。下面是一个典型的字符串的声明

```javascript
String s = "abc";
```

上面你创建了一个名为 `abc` 的字符串。

字符串是恒定的，一旦创建出来就不会被修改，怎么理解这句话？我们可以看下 String 源码的声明

![image-20211119103149924](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202111191031953.png)

String 对象是由`final` 修饰的，一旦使用 final 修饰的类不能被继承、方法不能被重写、属性不能被修改。而且 String 不只只有类是 final 的，它其中的方法也是由 final 修饰的，换句话说，Sring 类就是一个典型的 `Immutable` 类。也由于 String 的不可变性，类似字符串拼接、字符串截取等操作都会产生新的 String 对象。

下例分别创建了几个对象？

```java
String s1 = "aaa";
String s2 = "bbb" + "ccc";
String s3 = s1 + "bbb";
String s4 = new String("aaa");
```

- 首先第一个问题，s1 创建了几个对象。字符串在创建对象时，会在常量池中看有没有 aaa 这个字符串；如果没有此时还会在常量池中创建一个；如果有则不创建。我们默认是没有的情况，所以会创建一个对象。下同。

- 那么 s2 创建了几个对象呢？编译器做了优化 `String s2 = "bbb" + "ccc"` 会直接被优化为 `bbbccc`。也就是直接创建了一个 bbbccc 对象。

- s3 创建了几个对象呢？s3 执行 + 操作会创建一个 `StringBuilder` 对象然后执行初始化。执行 + 号相当于是执行 `new StringBuilder.append()` 操作。所以

  ```java
  String s3 = s1 + "bbb";
  
  ==
    
  String s3 = new StringBuilder().append(s1).append("bbb").toString();
  
  // Stringbuilder.toString() 方法也会创建一个 String 
  
  public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);
  }
  ```

  所以 s3 执行完成后，相当于创建了 3 个对象。（StringBuilder，"bbb", s3）

- 下面来看 s4 创建了几个对象，在创建这个对象时因为使用了 new 关键字，所以肯定会在堆中创建一个对象。然后会在常量池中看有没有 aaa 这个字符串；如果没有此时还会在常量池中创建一个；如果有则不创建。所以可能是创建一个或者两个对象，但是一定存在两个对象。

参考 https://www.cnblogs.com/leskang/p/6110631.html

## 如何理解 String 不可变

1. String 类使用 final 修饰，不可被继承
2. 在 jdk1.8 中，String 中使用 private final char[] value 存储元素，jdk9 之后使用 private final byte[] value 存储元素，这意味着 value 数组初始化之后就**不能再引用其它数组**，并且 String 类中没有提供任何可修改 value 数组的方法，从而保证 String 不可变。

## String 不可变的好处

1、**缓存 hash 值**

String 的 hash 值经常被使用，String 不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

2、**String 常量池的需要**

- 字符串的值存储值字符串池中，可以共享
- 字符串是常量，创建后不可改变。不可改变指的是，当改变字符串变量时，该变量内存地址将会重新指向在字符串池中重新开辟的内存空间的地址，字符串池中字符串的值不会改变。字符串池中的内容由JVM自动回收。

```java
package com.javaclass;

public class StringTest {
    public static void main(String[] args) {
        String s1 = "ABC";//"ABC"存储在字符串池中
        s1 = "EFG"; //给字符串赋值时，重新在字符串池中开辟内存空间存储"EFG"，栈中的name地址为"EFG"的地址
        String s2 = "ABC";
    }
}
```

![](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202112161521864.jpeg)

3、**安全性**

String 经常作为参数，String 不可变性可以保证参数不可变。

4、**线程安全**

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

## 如何使得 String 对象可变

首先通过源码，在 jdk8 中使用 private final char[] value 存储元素，因此直接访问不到 value，因此可通过反射进行改变

```java
public static void testReflection() throws Exception {
     
    //创建字符串"Hello World"， 并赋给引用s
    String s = "Hello World"; 
     
    System.out.println("s = " + s); //Hello World
     
    //获取String类中的value字段
    Field valueFieldOfString = String.class.getDeclaredField("value");
     
    //改变value属性的访问权限
    valueFieldOfString.setAccessible(true);
     
    //获取s对象上的value属性的值
    char[] value = (char[]) valueFieldOfString.get(s);
     
    //改变value所引用的数组中的第5个字符
    value[5] = '_';
     
    System.out.println("s = " + s);  //Hello_World
}
```

# 2、StringBuffer

`StringBuffer` 对象代表一个可变的字符串序列，当一个 StringBuffer 被创建以后，通过 StringBuffer 的一系列方法可以实现字符串的拼接、截取等操作。一旦通过 StringBuffer 生成了最终想要的字符串后，就可以调用其 `toString` 方法来生成一个新的字符串。

StringBuffer 是线程安全的，我们可以通过它的源码可以看出

![image-20211119103535821](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202111191035874.png)

StringBuffer 在字符串拼接上面直接使用 `synchronized` 关键字加锁，从而保证了线程安全性。

# 3、StringBuilder

StringBuilder 其实是和 StringBuffer 几乎一样，只不过 StringBuilder 是非线程安全的。并且，为什么 + 号操作符使用 StringBuilder 作为拼接条件而不是使用 StringBuffer 呢？我猜测原因是加锁是一个比较耗时的操作，而加锁会影响性能，所以 String 底层使用 StringBuilder 作为字符串拼接。

# 4、理解 String、StringBuilder、StringBuffer

StringBuilder 和 StringBuffer 都继承 AbstractStringBuilder，AbstractStringBuilder 是一个抽象类，里面定义了可变长字符串需要实现和使用的方法。

我们上面说到，使用 `+` 连接符时，JVM 会隐式创建 StringBuilder 对象，这种方式在大部分情况下并不会造成效率的损失，不过在进行大量循环拼接字符串时则需要注意。如下这段代码

```java
String s = "aaaa";
for (int i = 0; i < 100000; i++) {
    s += "bbb";
}
```

这是一段很普通的代码，只不过对字符串 s 进行了 + 操作，我们通过反编译代码来看一下。

```java
// 经过反编译后
String s = "aaa";
for(int i = 0; i < 10000; i++) {
     s = (new StringBuilder()).append(s).append("bbb").toString();    
}
```

你能看出来需要注意的地方了吗？在每次进行循环时，都会创建一个 `StringBuilder` 对象，每次都会把一个新的字符串元素 `bbb` 拼接到 `aaa` 的后面，所以，执行几次后的结果如下

![image-20211119103710156](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202111191037197.png)

每次都会创建一个 StringBuilder ，并把引用赋给 StringBuilder 对象，因此每个 StringBuilder 对象都是`强引用`， 这样在创建完毕后，内存中就会多了很多 StringBuilder 的无用对象。

这样由于大量 StringBuilder 创建在堆内存中，肯定会造成效率的损失，所以在这种情况下建议在循环体外创建一个 StringBuilder 对象调用 `append()`方法手动拼接。

例如

```javascript
StringBuilder builder = new StringBuilder("aaa");
for (int i = 0; i < 10000; i++) {
    builder.append("bbb");
}
builder.toString();
```

这段代码中，只会创建一个 builder 对象，每次循环都会使用这个 builder 对象进行拼接，因此提高了拼接效率。

> **StringBuilder**

StringBuilder 类表示一个可变的字符序列，我们知道，StringBuilder 是非线程安全的容器，一般适用于单线程场景中的字符串拼接操作，下面我们就来从源码角度看一下 StringBuilder

首先我们来看一下 StringBuilder 的定义

```java
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence {...}
```

StringBuilder 被 final 修饰，表示 StringBuilder 是不可被继承的，StringBuilder 类继承于 **AbstractStringBuilder**类。实际上，AbstractStringBuilder 类具体实现了可变字符序列的一系列操作，比如：append()、insert()、delete()、replace()、charAt() 方法等。

StringBuilder 实现了 2 个接口

- Serializable 序列化接口，表示对象可以被序列化。
- CharSequence 字符序列接口，提供了几个对字符序列进行只读访问的方法，例如 length()、charAt()、subSequence()、toString() 方法等。

StringBuilder 使用 AbstractStringBuilder 类中的两个变量作为元素

```java
char[] value; // 存储字符数组

int count; // 字符串使用的计数
```

> **StringBuffer**

StringBuffer 也是继承于 AbstractStringBuilder ，使用 value 和 count 分别表示存储的字符数组和字符串使用的计数，StringBuffer 与 StringBuilder 最大的区别就是 StringBuffer 可以在多线程场景下使用，StringBuffer 内部有大部分方法都加了 `synchronized` 锁。在单线程场景下效率比较低，因为有锁的开销。

# 5、StringBuilder 和 StringBuffer 的扩容问题

因为 StringBuilder 和 StringBuffer 均继承自 AbstractStringBuilder，二者 append 方法时，均调用父类的方法，因此扩容方法相同。

首先先注意一下 StringBuilder 的初始容量

```java
public StringBuilder() {
	super(16);
}
```

StringBuilder 的初始容量是 16，当然也可以指定 StringBuilder 的初始容量。

在调用 append 拼接字符串，会调用 AbstractStringBuilder 中的 append 方法

```java
public AbstractStringBuilder append(String str) {
  if (str == null)
    return appendNull();
  int len = str.length();
  ensureCapacityInternal(count + len);
  str.getChars(0, len, value, count);
  count += len;
  return this;
}
```

上面代码中有一个 `ensureCapacityInternal` 方法，这个就是扩容方法，我们跟进去看一下

```java
private void ensureCapacityInternal(int minimumCapacity) {
  // overflow-conscious code
  if (minimumCapacity - value.length > 0) {
    value = Arrays.copyOf(value,
                          newCapacity(minimumCapacity));
  }
}
```

这个方法会进行判断，minimumCapacity 就是字符长度 + 要拼接的字符串长度，如果拼接后的字符串要比当前字符长度大的话，会进行数据的复制，真正扩容的方法是在 `newCapacity` 中

```java
private int newCapacity(int minCapacity) {
  // overflow-conscious code
  int newCapacity = (value.length << 1) + 2;
  if (newCapacity - minCapacity < 0) {
    newCapacity = minCapacity;
  }
  return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
    ? hugeCapacity(minCapacity)
    : newCapacity;
}
```

扩容后的字符串长度是`原数组 value 长度增加一倍 + 2`，如果扩容后的长度还比拼接后的字符串长度小的话，那就直接扩容到它需要的长度  newCapacity = minCapacity，然后再进行数组的拷贝。

