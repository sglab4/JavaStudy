# 1、数据类型

## 基本类型

- byte/8 bit
- char/16 bit
- short/16 bit
- int/32 bit
- float/32 bit
- long/64 bit
- double/64 bit
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

在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 -XX:AutoBoxCacheMax=<size\> 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。

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
- 字符串是常量，创建后不可改变。不可改变指的是，当改变字符串变量时，该变量内存地址将会重新指向在字符串池中重新开辟的内存空间的地址，字符串池中字符串的值不会改变。字符串池中的内容由 JVM 自动回收。

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

![](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202203240940547.jpeg)

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
a++; //这里返回的 a 为 Short 类型，隐式类型转换
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

1、数据

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

```java
final int x = 1;  //x 值不能变
final A y = new A(); //y 不能改变引用
y.a = 1;
```

2、方法

声明方法不能被子类重写。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

3、类

被 final 声明的类不能被继承

## static

1、静态变量

- 静态变量：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。
- 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。

2、静态方法

静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它**不能是抽象方法**。

静态方法中只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，因为这两个关键字与具体对象关联。

3、静态语句块

静态语句块在类初始化时运行一次。

4、静态内部类

非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。

静态内部类不能访问外部类的非静态的变量和方法。

```java
public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}
```

5、静态导包

静态导包就是java包的静态导入，用import static代替import静态导入包

一般我们导入一个类都用 import com…..ClassName;而静态导入是这样：import static com…..ClassName.*;这里的多了个static，还有就是类名ClassName后面多了个 . * ，意思是导入这个类里的静态方法。当然，也可以只导入某个静态方法，只要把 .* 换成静态方法名就行了。然后在这个类中，就可以直接用方法名调用静态方法，而不必用ClassName.方法名 的方式来调用。

6、初始化顺序

静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。

顺序如下：

```java
//1
public static String staticField = "静态变量";
static {
    System.out.println("静态语句块");
}
//2
public String field = "实例变量";
//3
{
    System.out.println("普通语句块");
}
//4 最后是构造函数的初始化
public InitialOrderTest() {
    System.out.println("构造函数");
}
```

存在继承的情况下，初始化顺序为

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

# 5、Object 通用方法

## 概述

```java
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```

## equals()

### 1、等价关系

1. 自反性

   ```java
   x.equals(x); // true
   ```

2. 对称性

   ```java
   x.equals(y) == y.equals(x); // true
   ```

3. 传递性

   ```java
   if (x.equals(y) && y.equals(z))
       x.equals(z); // true;
   ```

4. 一致性

   多次调用 equals() 方法结果不变

   ```java
   x.equals(y) == x.equals(y); // true
   ```

5. 与 null 的比较

   对任何不是 null 的对象 x 调用 x.equals(null) 结果都为 false

   ```java
   x.equals(null); // false;
   ```

### 2、== 和 equals

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否引用同一个对象，对于已经重写 equals() 的类，equals() 判断引用的对象是否等价。而未重写 equals() 的类，由于所有类均继承 Object，Object equals() 源码如下，未重写 equals() 的类判断的是两个变量是否引用同一个对象。

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

```java
public class Test {
    public static void main(String[] args) {
        Integer x = new Integer(1); 
        Integer y = new Integer(1);
        System.out.println(x == y); //false
        System.out.println(x.equals(y)); //true
        y = x;
        System.out.println(x == y); //true

        A a = new A("a", 1);
        A b = new A("a", 1);
        System.out.println(a == b); //false
        System.out.println(a.equals(b)); //false
    }
}

class A{
    String name;
    int age;

    public A(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

### 3、重写 equals()

4 个步骤

- 检查是否为同一个对象的引用，如果是直接返回 true；
- 检查是否是同一个类型，如果不是，直接返回 false；
- 将 Object 对象进行转型；
- 判断每个关键域是否相等。

```java
class A{
    String name;
    int age;

    public A(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        A a = (A) o;
        return age == a.age && name.equals(a.name);
    }
}
```

## hashCode()

hashCode() 返回对象的哈希值（散列值）。等价的两个对象的哈希值一定相同，而哈希值相同的两个对象却不一定等价，这是因为哈希值的计算具有随机性，因此不等价的对象可能计算出相同的哈希值。

**通常在重写 equals() 的时候也会重写 hashCode() 方法，来保证等价的两个对象哈希值相同。**

HashSet 和 HashMap 等集合类使用了 hashCode() 方法来计算对象应该存储的位置，因此要将对象添加到这些集合类中，需要让对应的类实现 hashCode() 方法。

以下代码中，EqualExample 重写 hashCode() 方法，因此即使 e1 和 e2 等价，但是因为没有重写 hashCode() 方法，导致在 HashSet 中加入了两个等价的对象。

```java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2
```

理想的哈希函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的哈希值上。这就要求了哈希函数要把所有域的值都考虑进来。可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。

R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位，最左边的位丢失。并且一个数与 31 相乘可以转换成移位和减法：`31 * x == (x << 5) - x`，编译器会自动进行这个优化。

```java
//实例变量为 x, y, z
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```

## toString()

默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面的数值为哈希值的无符号十六进制表示。

```
A a1 = new A("a1", 2);
System.out.println(Integer.toString(a1.hashCode(), 16));
System.out.println(a1);
```

输出：

```html
175d3
com.leetcode.A@175d3
```

## clone()

### 1、Cloneable

clone() 是 Object 的 protected 方法，若一个类不显式去重写 clone()，那么这个类的实例便不能调用 clone() 方法

```java
class CloneExample {
    private int a;
    private int b;
}
```

```java
CloneExample e1 = new CloneExample();
// CloneExample e2 = e1.clone(); // 出错，'clone()' has protected access in 'java.lang.Object'
```

重写 clone() 方法

```java
class CloneExample{
    private int x;
    private int y;

    @Override
    protected CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample) super.clone();
    }
}
```

```java
CloneExample ce1 = new CloneExample();
try {
    CloneExample ce2 = ce1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
```

再次运行，发现抛出异常

![image-20210521204050557](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210521204057.png)

这是因为 CloneExample 类未实现 Cloneable 接口，对 CloneExample 进行修改

```java
class CloneExample implements Cloneable {
    private int x;
    private int y;

    @Override
    protected CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample) super.clone();
    }
}
```

再次运行，将不会出错。打开 Cloneable 源码，如下

```
public interface Cloneable {
}
```

可发现，clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是标识规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。

### 2、浅拷贝

拷贝对象和原始对象的引用类型**引用同一个对象**。**浅拷贝只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存。**

> 赋值和浅拷贝的区别

把一个对象赋值给一个新的变量时，**赋的其实是该对象的在栈中的地址，而不是堆中的数据**。也就是两个对象指向的是同一个存储空间，无论哪个对象发生改变，其实都是改变的存储空间的内容，因此，两个对象是联动的。

浅拷贝是按位拷贝对象，**它会创建一个新对象**，这个对象有着原始对象属性值的一份精确拷贝。如果属性是**基本类型**，拷贝的就是基本类型的值，改变一个对象不会对另一个对象造成影响；如果属性是**内存地址（引用类型）**，拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。

如下所示，当浅拷贝 CloneExample 类时，改变一个对象的 arr 将会对另一个对象造成影响；改变一个对象的 x 或 y 时，将不会影响另一个对象的内容。

```java
class CloneExample implements Cloneable {
    private int[] arr;
    private int x;
    private int y;

    @Override
    protected CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample) super.clone();
    }
}
```

### 3、深拷贝

拷贝对象和原始对象的引用类型引用不同对象。深拷贝会另外 new 一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象。

```java
class CloneExample implements Cloneable {
    private int[] arr;
    private int x;
    private int y;

    @Override
    protected CloneExample clone() throws CloneNotSupportedException {
        CloneExample result = new CloneExample();
        //对成员变量初始化
        result.arr = new int[10];
        result.x = 1;
        result.y = 1;
        return result;
    }
}
```

### 4、clone() 的替代方案

使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用**拷贝构造函数或者拷贝工厂**来拷贝一个对象。使用深拷贝的方法，新对象跟原对象不共享内存，修改新对象不会改到原对象。

```java
class CloneConstructorExample{
    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < 10; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = i;
        }
    }

    public void setArr(int index, int value) {
        arr[index] = value;
    }

    public int getArr(int index) {
        return arr[index];
    }
}
```

```java
CloneConstructorExample cce1 = new CloneConstructorExample();
CloneConstructorExample cce2 = new CloneConstructorExample(cce1);
cce1.setArr(2, 222);
System.out.println(cce1.getArr(2));
System.out.println(cce2.getArr(2));
```

输出：

```html
222
2
```

# 6、继承

## 访问权限

Java 中有四个访问权限修饰符 private、protected、默认、public，默认修饰符仅包级可见。可以对类或类中的成员（字段和方法）加上访问修饰符。

- 类可见表示其它类可以用这个类创建实例对象。
- 成员可见表示其它类可以用这个类的实例对象访问到该成员；

protected 用于修饰成员，表示在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义。

设计良好的模块会隐藏所有的实现细节，把它的 API 与它的实现清晰地隔离开来。模块之间只通过它们的 API 进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念被称为**信息隐藏或封装**。因此访问权限应当尽可能地使每个类或者成员不被外界访问。

如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例去代替，也就是确保满足里氏替换原则。

里氏替换原则通俗的来讲就是：**子类可以扩展父类的功能，但不能改变父类原有的功能。**它包含以下4层含义：

- 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法，这句话应该理解为“子类重写父类方法时，可以改变方法的具体行为，但不应该改变方法的用途”
- 子类中可以增加自己特有的方法。
- 当子类的方法重载父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松。
- 当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格。

```java
父类 变量名 = new 子类();
```

在上述代码中，**该变量名仅能看到父类中的所有方法，对于子类中新增的方法，父类不可使用。子类对父类中的方法重写后，输出的为子类中重写的方法。**

在 Java 封装原则中，字段决不允许是共有的，因为这样的话就失去了对这个字段修改行为的控制，客户端可以对其随意修改。可以使用公有的 getter 和 setter 方法来替换公有字段，这样的话就可以控制对字段的修改行为。

```java
public class AccessExample {

    private int id;

    public String getId() {
        return id + "";
    }

    public void setId(String id) {
        this.id = Integer.valueOf(id);
    }
}
```

如果是包级私有的类或者私有的嵌套类，那么直接暴露成员不会有特别大的影响。

```java
public class AccessWithInnerClassExample {

    private class InnerClass {
        int x;
    }

    private InnerClass innerClass;

    public AccessWithInnerClassExample() {
        innerClass = new InnerClass();
    }

    public int getValue() {
        return innerClass.x;  // 直接访问
    }
}
```

##  抽象类与接口

### 1、抽象类

抽象类和抽象方法都使用 abstract 关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。抽象类中也可以有普通方法和成员变量，也可以没有抽象方法。

抽象类和普通类最大的区别是，抽象类不能被实例化，只能被继承。

```java
abstract class abstractClass {

    private int a;
    private String b;

    protected abstract void fun();

    public void out() {
        System.out.println("abstractClass");
    }
}

class extendsClass extends abstractClass {

    @Override
    protected void fun() {
        System.out.println("fun");
    }
}
```

可使用匿名内部类和继承抽象类的方法实现抽象方法。

```java
abstractClass a1 = new abstractClass() {
    @Override
    protected void fun() {
        System.out.println("a1");
    }
};
a1.fun();
extendsClass e1 = new extendsClass();
e1.fun();
```

### 2、接口

接口是抽象类的延伸，接口中所有的成员（变量和方法）默认都是 public，不允许定义为 private 或者 protected。对于成员变量，默认都是 static 和 final 的，并且需要初始化给定一个值。

接口中也可定义 default 类型的方法，并且需要在接口中实现，不允许定义 default 类型的成员变量。

在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。从 Java 8 开始，接口也可以拥有默认（default）的方法实现，从 Java 9 开始，允许将方法定义为 private，这样就能定义某些复用的代码又不会把方法暴露出去。

### 3、抽象类和接口的比较

- 从设计层面上看，抽象类提供了一种 IS-A 关系，需要满足里式替换原则，即子类对象必须能够替换掉所有父类对象。而接口更像是一种 LIKE-A 关系，它只是提供一种方法实现契约，并不要求接口和实现接口的类具有 IS-A 关系。
- 从使用上来看，一个类可以实现多个接口，但是不能多继承
- 接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。
- 接口的成员只能是 public 的（Java 8 开始也可定义 default 方法，Java 9 之后允许将方法定义为 private），而抽象类的成员可以有多种访问权限。

### 4、使用选择

接口：

- 需要让不相关的类都实现一个方法，例如不相关的类都可以实现 Comparable 接口中的 compareTo() 方法；
- 需要使用多继承，一个类实现多个接口。

抽象类：

- 需要在几个相关的类中共享代码。
- 需要能控制继承来的成员的访问权限，而不是都为 public。
- 需要继承非静态和非常量字段，因为接口中的字段均为 static 和 final。

在很多情况下，接口优先于抽象类。因为接口没有抽象类严格的类层次结构要求，可以灵活地为一个类添加行为。并且从 Java 8 开始，接口也可以有默认的方法实现，使得修改接口的成本也变的很低。

## super

- 访问父类的构造函数：使用 super() 访问的就是父类的构造函数，使用父类的构造函数完成一些初始化工作。应该注意到，子类在构造过程中会首先调用父类默认的构造函数，若想调用父类其他带参数的构造函数，需要使用 super(参数) 进行调用，然后再调用子类的构造函数

```java
class Person {
    Person() {
        System.out.println("父类构造函数");
    }
    Person(int a) {
        System.out.println(a);
    }
}

class student extends Person {
    student() {
        System.out.println("子类构造函数");
    }
}
```

```java
Person person = new student();
```

输出：

```html
父类构造函数
子类构造函数
```

```java
class Person {
    Person() {
        System.out.println("父类构造函数");
    }
    Person(int a) {
        System.out.println(a);
    }
}

class student extends Person {
    student() {
        super(1);
        System.out.println("子类构造函数");
    }
}
```

```java
Person person = new student();
```

输出：

```html
1
子类构造函数
```

- 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现。

```java
class Person {
    protected void out() {
        System.out.println("Person");
    }
}

class student extends Person {
    @Override
    protected void out() {
        super.out();
        System.out.println("student");
    }
}
```

输出：

```html
Person
student
```

## 重写与重载

### 1、重写（Override）

存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法。

- 子类方法的访问权限必须大于等于父类方法；
- 子类方法的返回类型必须是父类方法返回类型或为其子类型。
- 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。

使用 @Override 注解，可以让编译器帮忙检查是否满足上面的三个限制条件。

```java
class Person {
    protected List<Integer> out() {
        List<Integer> list = new ArrayList<>();
        return list;
    }
}

class student extends Person {
    @Override
    public ArrayList<Integer> out() {
        ArrayList<Integer> arrayList = new ArrayList<>();
        return arrayList;
    }
}
```

在调用一个方法时，先从本类中查找看是否有对应的方法，如果没有再到父类中查看。总的来说，方法调用的优先级为：

- this.func(this)
- super.func(this)
- this.func(super)
- super.func(super)

```java
class A {

    public void show(A obj) {
        System.out.println("A.show(A)");
    }

    public void show(C obj) {
        System.out.println("A.show(C)");
    }
}

class B extends A {

    @Override
    public void show(A obj) {
        System.out.println("B.show(A)");
    }
}

class C extends B {
}

class D extends C {
}
```

```java
public static void main(String[] args) {

    A a = new A();
    B b = new B();
    C c = new C();
    D d = new D();

    // 在 A 中存在 show(A obj)，直接调用
    a.show(a); // A.show(A)
    // 在 A 中不存在 show(B obj)，将 B 转型成其父类 A
    a.show(b); // A.show(A)
    // 在 B 中存在从 A 继承来的 show(C obj)，直接调用
    b.show(c); // A.show(C)
    // 在 B 中不存在 show(D obj)，但是存在从 A 继承来的 show(C obj)，将 D 转型成其父类 C
    b.show(d); // A.show(C)

    // 引用的还是 B 对象，所以 ba 和 b 的调用结果一样
    A ba = new B();
    ba.show(c); // A.show(C)
    ba.show(d); // A.show(C)
}
```

### 2、重载（Overload）

存在于**同一个类中**，指一个方法与已经存在的方法名称上相同，但是**参数类型、个数、顺序**至少有一个不同。

注意：只有返回值不同，其他所有均相同的两个方法不算重载，会报错。

```java
class Person {
    //出错
    /*protected void show() {
        System.out.println("Person Show");
    }*/
	//出错
    /*protected int show() {
        return 1;
    }*/
}
```

```java
class Person {
    protected void show() {
        System.out.println("Person Show");
    }
	//重载
    protected void show(String a) {
        System.out.println("Person Show" + a);
    }
}
```

# 7、反射

**每个类**都有一个 **Class** 对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。

类加载相当于 Class 对象的加载，类在第一次使用时才动态加载到 JVM 中。也可以使用 `Class.forName("com.mysql.jdbc.Driver")` 这种方式来控制类的加载，该方法会返回一个 Class 对象。

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的 .class 不存在也可以加载进来。

Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

- **Field** ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
- **Method** ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
- **Constructor** ：可以用 Constructor 的 newInstance() 创建新的对象。

反射的优点：

- **可扩展性** ：应用程序可以利用全限定名创建可扩展对象的实例，来使用来自外部的用户自定义类。
- **类浏览器和可视化开发环境** ：一个类浏览器需要可以枚举类的成员。可视化开发环境（如 IDE）可以从利用反射中可用的类型信息中受益，以帮助程序员编写正确的代码。
- **调试器和测试工具** ：调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动地调用类里定义的可被发现的 API 定义，以确保一组测试中有较高的代码覆盖率。

反射的缺点：

尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。

- **性能开销** ：反射涉及了动态类型的解析，所以 JVM 无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。
- **安全限制** ：使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如 Applet，那么这就是个问题了。
- **内部暴露** ：由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。

# 8、异常

见 https://www.cnblogs.com/sgKurisu/p/14406588.html

Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： **Error** 和 **Exception**。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种

- **受检异常** ：粉红色部分，需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；
- **非受检异常** ：绿色部分是运行时异常，是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复。

![image-20210806141336564](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210806141337.png)

常见的OOM异常（错误）有以下6种。 

- OutOfMemoryError:Java heap space 堆内存不够
- OutOfMemoryError:GC overhead limit exceeded GC回收时间过长，过长是指98%的时间用来GC却回收不到2%的堆内存
- OutOfMemoryError:Direct buffer memory 堆内存充足但本地内存可能已经用完
- OutOfMemoryError:unable to create new native thread 创建了太多线程，超过了系统承载极限
- OutOfMemoryError:Metaspace 元空间内存不够
- StackOverflowError 栈内存不够

# 9、泛型

见博客 https://www.cnblogs.com/sgKurisu/p/14488659.html

泛型的本质是参数化类型，即把类型作为参数传递

```java
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

## 泛型擦除

参考 https://www.cnblogs.com/wuqinglong/p/9456193.html

# 10、注解

Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。

> Override 注解

指明被注解的方法需要覆写超类中的方法，如果某个方法使用了该注解,却没有覆写超类中的方法(比如大小写写错了,或者参数错了,或者是子类自己定义的方法),编译器就会生成一个错误。

> Deprecated 注解

可以修饰类、方法、变量，在java源码中被@Deprecated修饰的类、方法、变量等表示不建议使用的，可能会出现错误的，可能以后会被删除的类、方法等，如果现在使用，则在以后使用了这些类、方法的程序在更新新的JDK、jar包等就会出错，不再提供支持。

个人程序中的类、方法、变量用@Deprecated修饰同样是不希望自己和别人在以后的时间再次使用此类、方法。 当编译器编译时遇到了使用@Deprecated修饰的类、方法、变量时会提示相应的警告信息。

> Suppresswarnings 注解

可以达到抑制编译器编译时产生警告的目的，但是很不建议使用@SuppressWarnings注解，使用此注解，编码人员看不到编译时编译器提示的相应的警告，不能选择更好、更新的类、方法或者不能编写更规范的编码。同时后期更新JDK、jar包等源码时，使用@SuppressWarnings注解的代码可能受新的JDK、jar包代码的支持，出现错误，仍然需要修改。

| 关键字                   | 用途                                                   |
| ------------------------ | ------------------------------------------------------ |
| all                      | 抑制所有警告                                           |
| boxing                   | 抑制装箱、拆箱操作时候的警告                           |
| cast                     | 抑制映射相关的警告                                     |
| dep-ann                  | 抑制启用注释的警告                                     |
| deprecation              | 抑制过期方法警告                                       |
| fallthrough              | 抑制在 switch 中缺失 breaks 的警告                     |
| finally                  | 抑制 finally 模块没有返回的警告                        |
| hiding                   | 抑制相对于隐藏变量的局部变量的警告                     |
| incomplete-switch        | 忽略不完整的 switch 语句                               |
| nls                      | 忽略非 nls 格式的字符                                  |
| null                     | 忽略对 null 的操作                                     |
| rawtypes                 | 使用 generics 时忽略没有指定相应的类型                 |
| restriction              | 抑制禁止使用劝阻或禁止引用的警告                       |
| serial                   | 忽略在 serializable 类中没有声明 serialVersionUID 变量 |
| static-access            | 抑制不正确的静态访问方式警告                           |
| synthetic-access         | 抑制子类没有按最优方法访问内部类的警告                 |
| unchecked                | 抑制没有进行类型检查操作的警告                         |
| unqualified-field-access | 抑制没有权限访问的域的警告                             |
| unused                   | 抑制没被使用过的代码的警告                             |

## Java 注解的实现原理

参考 https://www.cnblogs.com/yangming1996/p/9295168.html

**注解的本质就是一个继承了 Annotation 接口的接口**，一个注解准确意义上来说，只不过是一种特殊的注释而已，解析一个类或者方法的注解往往有两种形式，一种是编译期直接的扫描，一种是运行期反射。

**编译器的扫描**指的是编译器在对 java 代码编译字节码的过程中会检测到某个类或者方法被一些注解修饰，这时它就会对于这些注解进行某些处理。典型的就是注解 @Override，一旦编译器检测到某个方法被修饰了 @Override 注解，编译器就会检查当前方法的方法签名是否真正重写了父类的某个方法，也就是比较父类中是否具有一个同样的方法签名。

这一种情况只适用于那些编译器已经熟知的注解类，比如 JDK 内置的几个注解，而你自定义的注解，编译器是不知道你这个注解的作用的，当然也不知道该如何处理，往往只是会根据该注解的作用范围来选择是否编译进字节码文件，仅此而已。

### 元注解

JAVA 中有以下几个『元注解』：

- @Target：注解的作用目标
- @Retention：注解的生命周期
- @Documented：注解是否应当被包含在 JavaDoc 文档中
- @Inherited：是否允许子类继承该注解

### 内置注解

- @Override
- @Deprecated
- @SuppressWarnings

### 运行期间反射

我们说过，注解本质上是继承了 Annotation 接口的接口，而当你通过反射，也就是我们这里的 getAnnotation 方法去获取一个注解类实例的时候，其实 JDK 是通过动态代理机制生成一个实现我们注解（接口）的代理类。

![image](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209222105817.png)

![image](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204061626892.png)

![image](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204061627820.png)

![image-20220922210604765](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209222106838.png)

代理类实现接口 Hello 并重写其所有方法，包括 value 方法以及接口 Hello 从 Annotation 接口继承而来的方法。

运行期间反射实现注解的具体方法是 Java 运行时生成的动态代理类。而我们通过反射获取注解时，返回的是Java 运行时生成的动态代理对象$Proxy1。通过代理对象调用自定义注解（接口）的方法，会最终调用AnnotationInvocationHandler 的invoke 方法。invoke 首先匹配看当前调用的方法是否为 toString，equals，hashCode，annotationType，若是，则直接返回方法的实现，否则从memberValues 这个Map 中索引出对应的值进行返回。而memberValues 的来源是Java 常量池。

那么这样，一个注解的实例就创建出来了，它本质上就是一个代理类， AnnotationInvocationHandler 中 invoke 方法的实现逻辑，这是核心。一句话概括就是，**通过方法名返回注解属性值**。

![image-20220806163650794](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061636872.png)

![image-20220806163709343](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061637414.png)

# 11、特性

## Java 各版本新特性

参考 https://www.cnblogs.com/javazhiyin/p/11394448.html

Java 各版本发布时间

![](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210524192803.png)

### Java 8

**1、Lambda 和 函数式接口**

**2、方法引用**

- 构造方法引用
- 静态方法引用
- 对象的实例方法引用
- 类的实例方法引用

**3、接口默认方法和静态方法**

```java
public interface TestInterface {
    String test();

    // 接口默认方法
    default String defaultTest() {
        return "default";
    }

    static String staticTest() {
        return "static";
    }
}
```

**4、重复注解**

**5、类型注解**

**6、更好的类型推断**

Java 7 中如下

```java
List<String> stringList = new ArrayList<>();
stringList.add("A");
stringList.addAll(Arrays.<String>asList());
```

在 Java 8 中

```java
List<String> stringList = new ArrayList<>();
stringList.add("A");
stringList.addAll(Arrays.asList());
```

**7、Optional**

Java 8 中新增了 Optional 类用来解决空指针异常。Optional 是一个可以保存 null 的容器对象。通过 isPresent() 方法检测值是否存在，通过 get() 方法返回对象。

**8、Stream**

Java 8 中新增的 Stream 类提供了一种新的数据处理方式。这种方式将元素集合看做一种流，在管道中传输，经过一系列处理节点，最终输出结果。

**9、日期时间 API**

Java 8 中新增了日期时间 API 用来加强对日期时间的处理，其中包括了 LocalDate，LocalTime，LocalDateTime，ZonedDateTime 等等

可参考 https://www.cnblogs.com/muscleape/p/9956754.html

**10、Base64 支持**

Java 8 标准库中提供了对 Base 64 编码的支持。

**11、并行数组 ParallelSort**

Arrays.parallelSort

### Java 9

**1、Jigsaw 模块系统**

**2、JShell REPL**

Java 9 提供了交互式解释器。可在 Shell 中运行一些代码并直接得出结果。

**3、私有接口方法，接口中使用私有方法**

```java
public interface TestInterface {
    String test();

    // 接口默认方法
    default String defaultTest() {
        pmethod();
        return "default";
    }

    private String pmethod() {
        System.out.println("private method in interface");
        return "private";
    }
}
```

**4、集合不可变实例工厂方法**

**5、改进 try-with-resources**

**6、多版本兼容 jar 包**

**7、增强了 Stream，Optional，Process API**

**8、新增 HTTP2 Client**

**9、增强 Javadoc，增加了 HTML 5 文档的输出，并且增加了搜索功能**

### Java 10

**1、新增局部类型推断 var**

var 关键字目前只能用于局部变量以及 for 循环变量声明中。

**2、删除工具 javah**

**3、统一的垃圾回收接口，改进了 GC 和其他内务管理**

### Java 11

**1、Lambda 中使用 var**

**2、java 直接编译并运行，省去先 javac 编译生成 class 再运行的步骤**

**3、增加对 TLS 1.3 的支持**

### Java 12

**1、switch 表达式**

Java 12 以后，switch 不仅可以作为语句，也可以作为表达式。

```java
private String switchTest(int i) {
    return switch (i) {
        case 1 -> "1";
        default -> "0";
    };
}
```

![](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210524195051.jpeg)

## Java 和 C++ 区别

- Java 是纯粹的面向对象语言，所有的对象都继承自 java.lang.Object，C++ 为了兼容 C 既支持面向对象也支持面向过程。
- Java 通过虚拟机从而实现跨平台特性，但是 C++ 依赖于特定的平台。
- Java 没有指针，它的引用可以理解为安全指针，而 C++ 具有和 C 一样的指针。
- Java 支持自动垃圾回收，而 C++ 需要手动回收。
- Java 不支持多重继承，只能通过实现多个接口来达到相同目的，而 C++ 支持多重继承。
- Java 不支持操作符重载，虽然可以对两个 String 对象执行加法运算，但是这是语言内置支持的操作，不属于操作符重载，而 C++ 可以。
- Java 的 goto 是保留字，但是不可用，C++ 可以使用 goto。

## JDK 和 JRE

- JRE：Java Runtime Environment，Java 运行环境的简称，为 Java 的运行提供了所需的环境。它是一个 JVM 程序，主要包括了 JVM 的标准实现和一些 Java 基本类库。
- JDK：Java Development Kit，Java 开发工具包，提供了 Java 的开发及运行环境。JDK 是 Java 开发的核心，集成了 JRE 以及一些其它的工具，比如编译 Java 源码的编译器 javac 等。

