参考 http://www.cyc2018.xyz/

# 1、数据类型

## 基本类型

- byte/8
- char/16
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

boolean 只有两个值：true、false，boolean 类型所占空间不确定，可能为

1. 1 个 bit

   理由是boolean类型的值只有true和false两种逻辑值，在编译后会使用1和0来表示，这两个数在内存中只需要1位（bit）即可存储，位是计算机最小的存储单位。

2. 1 个字节

   计算机处理数据的最小单位是1个字节，1个字节等于8位，实际存储的空间是：用1个字节的最低位存储，其他7位用0填补，如果值是true的话则存储的二进制为：0000 0001，如果是false的话则存储的二进制为：0000 0000。

3. 4 个字节

   根据《Java虚拟机规范》"Java语言表达式所操作的boolean值，在编译之后都使用Java虚拟机中的int数据类型来代替以"，得出boolean类型占了单独使用是4个字节，在数组中又是1个字节。

所以1个字节、4个字节都是有可能的。这其实是运算效率和存储空间之间的博弈，两者都非常的重要。

JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。

## 包装类型

JDK 1.5 之后，基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成

```java
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 x.intValue()
```

## 缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

```java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

valueOf() 方法的源码：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

当 valueOf(x) 时，先判断是否在缓存池中（也就是判断 x 是否在 -128~127 之间），若是，则将堆空间中的地址直接返回给栈空间，否则重新开辟一个内存空间。具体可见  https://www.cnblogs.com/sgKurisu/p/14431755.html

```java
    Integer integer3 = Integer.valueOf(100);
    Integer integer4 = Integer.valueOf(100);
    System.out.println(integer3 == integer4); //true

    /*Integer integer5 = 200;
    Integer integer6 = 200;*/
    Integer integer5 = Integer.valueOf(200);
    Integer integer6 = Integer.valueOf(200);
    System.out.println(integer5 == integer6); //false
```

**基本类型对应的缓冲池如下**：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象（即地址相等）。

在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 -XX:AutoBoxCacheMax=<size> 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。

# 2、String

## 概述

String 被声明为 final，因此它不可被继承。(Integer 等包装类也不能被继承）

在 Java 8 中，String 内部使用 char 数组存储数据。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

value 数组被声明为 final，这意味着 value 数组初始化之后就**不能再引用其它数组**。并且 String 内部没有改变 value 数组的方法，因此可以保证 **String 不可变** 

## 不可变的好处

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



![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210221100944%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%86%85%E5%AD%98%E5%88%86%E6%9E%90.jpg)

3、**安全性**

String 经常作为参数，String 不可变性可以保证参数不可变。

4、**线程安全**

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

## String, StringBuffer and StringBuilder

1、**可变性**

- String 不可变
- StringBuffer 和 StringBuilder 可变，使用 append 方法在末尾添加新的字符串

2、**线程安全**

- String 不可变，是线程安全的

- StringBuffer：可变长字符串，效率低，线程安全，使用 synchronized 同步
- StringBuilder：可变长字符串，效率高，线程不安全

## String Pool

字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。字面量指的是例如 String s = "abc", "abc" 为字面量。还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

如下代码，先对 s1 和 s2 开辟不同内存空间，在第三行比较时，内存地址不同，因此返回 false。而 s3 和 s4 是通过 s1.intern() 和 s2.intern() 方法取得同一个字符串引用，返回 true。

```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s2.intern();
System.out.println(s3 == s4);           // true
```

但是，若采用以下形式创建字符串，将会被直接放入到 String Pool 中

```java
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6);  // true
```

> 总结

采用 String a = new String("abc")，"abc" 不会被直接放到字符串常量池（String Pool） 中，那么当两个字符串采用这种方式定义时，即使 "abc" 内容相同，s1 == s2  将返回 false。

若采用 String a = "abc"，"abc" 将会被直接放到常量池中，a 的地址为 "abc" 常量字符串的地址， s1 == s2 将会返回 true。

## new String("abc")

使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "abc" 字符串对象）。

- 一个是 String Pool 中的字符串对象，"abc" 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "abc" 字符串字面量；
- 另一个是使用 new 命令在堆中创建一个字符串对象

```java
public class NewStringTest {
    public static void main(String[] args) {
        String s = new String("abc");
    }
}
```

对上述代码使用 javap -verbose 反编译，得到

```java
// ...
Constant pool:
// ...
   #2 = Class              #18            // java/lang/String
   #3 = String             #19            // abc
// ...
  #18 = Utf8               java/lang/String
  #19 = Utf8               abc
// ...

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #2                  // class java/lang/String
         3: dup
         4: ldc           #3                  // String abc
         6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
         9: astore_1
// ...
```

在 Constant Pool 中，#19 存储这字符串字面量 "abc"，#3 是 String Pool 的字符串对象，它指向 #19 这个字符串字面量。在 main 方法中，0: 行使用 new #2 在堆中创建一个字符串对象，并且使用 ldc #3 将 String Pool 中的字符串对象**作为 String 构造函数的参数**。

String 构造函数如下：

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

可发现，将一个字符串对象作为另一个字符串的构造参数时，并不会复制 value（指的是 byte 数组，可查看源码） 中的内容，而是会指向同一个 value 数组。

# 3、运算

## 参数传递

Java 的参数是以**值传递**的形式传入方法中，而不是引用传递。

以下代码中，使用 test 方法传递的是值，也就是将 s1 中的内容 "abc" 传给 s（可理解为 s1 作为了 s 的构造参数），s1 和 s 的内存地址不同，因此改变 s 并不会影响到 s1

```java
public class Test {
    public static void main(String[] args) {
        String s1 = new String("abc");
        System.out.println(s1); //"abc"
        new Test().test(s1);
        System.out.println(s1); //"abc"
    }

    private void test(String s) {
        s = "123";
    }
}
```

下面自定义类 Dog

```java
public class Dog {

    String name;

    Dog(String name) {
        this.name = name;
    }

    String getName() {
        return this.name;
    }

    void setName(String name) {
        this.name = name;
    }

    String getObjectAddress() {
        return super.toString();
    }
}
```

Dog dog，其中 dog 为一个指针，存储的是对象的内存地址，当调用 func 方法时，本质上是将 dog 的地址以值的方式传递到形参中，因此在 func 方法中进行改变，将会影响原本的内容。

```java
class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        func(dog);
        System.out.println(dog.getName());          // B
    }

    private static void func(Dog dog) {
        dog.setName("B");
    }
}
```

如下代码，若在方法中将指针指向了其他对象，那么再对指针中的内容变化时，将不会影响到原对象。

```java
public class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        func(dog);
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        System.out.println(dog.getName());          // A
    }

    private static void func(Dog dog) {
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        dog = new Dog("B");
        System.out.println(dog.getObjectAddress()); // Dog@74a14482
        System.out.println(dog.getName());          // B
    }
}
```

## float 与 double

Java 不能隐式执行向下转型，因为这会使得精度降低。需要进行强制转换

```java
float f = 1.1; //出现错误
```

这里的 1.1 字面量是 double 类型，需要强制转换 float f = (float) 1.1;  

1.1f 字面量才是 float 类型的变量

```java
float f = 1.1f;
```

## 隐式类型转换

这里以 short 和 int 的类型转换为例。参考 https://blog.csdn.net/allenjay11/article/details/78613862

1、由于 a + 1 为 int 类型，而 a 为 short 类型，因此将会编译失败

```java
public class HelloWord {
    //执行结果是？
    public static void main(String[] args){
       short a = 1; // 1 在这里是以 short 类型出现的
        a = a + 1;
       System.out.print(a);
    }
}
```

2、将会编译通过，因为 += 或者 ++ 将会进行隐式类型转换

```java
public class HelloWord {
    public static void main(String[] args){
       short a = 1;
        a += 1;
       System.out.print(a);
    }
}
```

```java
a += 1;
a++;
```

相当于 s1 = (short) (s1 + 1);向下转换，返回的是 short 类型

3、这样同样会编译失败，因为在运算中，精度小于 int 类型的运算将会被自动转换成 int 后进行操作

```java
public class HelloWord {
    public static void main(String[] args){
       short a = 1;
       short b = 1;
       a = a + 1; //这里将会出错，返回的是 int 类型
       short c = a + b;
       System.out.print(c);
    }
}
```

==**注意**：若以上定义为 Short 的包装类==

```java
Short a = 1;
a = a + 1; //出错，这里返回的是 int 类型
a += 1; //将会出错，这里返回的是 int 类型，和 short 不同的地方
a++; //这里返回的 a 仍未 Short 类型，隐式类型转换
```

## switch

从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象。switch 不支持 long、float、double，是因为 switch 的设计初衷是对那些只有少数几个值的类型进行等值判断，如果值过于复杂，那么还是用 if 比较合适。

```java
String s = "a";
switch (s) {
    case "a":
        System.out.println("aaa");
        break;
    case "b":
        System.out.println("bbb");
        break;
}
```

# 4、关键字

## final









