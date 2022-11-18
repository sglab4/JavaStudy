[TOC]

# 集合的概念

对象的容器，实现了对多个对象进行操作的常用方法，可实现数组的功能

和数组的区别：

- 数组长度固定，集合长度不固定
- 数组可存储基本类型和引用类型，集合仅可存储引用类型（基本类型进行装箱）

# Collection接口

Collection 层次结构中的根接口，Collection 继承了 Iterable 接口

![](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202210142008055.jpeg)

![](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211021160511.png)

常用方法：

```java
boolean add(E e) //添加对象
boolean addAll(Collection c)//将一个集合中的所有元素都添加到此集合中 
void clear()  //移除此 collection 中的所有元素
boolean contains(Object o) //判断此集合是否包含指定元素
boolean equals(Object o) //比较此集合与指定对象是否相等
boolean isEmpty()  //判断此集合是否为空
boolean remove(Object o) //从此集合中移除指定元素
int size()//返回此集合中的元素数 
Object[] toArray()//将此集合转化为数组
Iterator<E> iterator() //返回在此集合的元素上进行迭代的迭代器。用来遍历集合
    				   //Iterator为接口
```

做删除操作时，删除的为地址，而非实例对象

Iterator接口：

对 collection 进行迭代的迭代器。在迭代过程中，不允许使用Collection中的方法删除，会出现并发异常

```java
boolean hasNext() //如果仍有元素可以迭代，则返回 true，否则返回false 
E next()//获取下一个元素。 
void remove()//删除当前元素 
```

```java
package com.sgkurisu.collection;

import java.util.ArrayList;
import java.util.Collection;
import java.util.Iterator;

//Collection测试
public class TestCollection {
    public static void main(String[] args) {
        Collection collection = new ArrayList();

        //添加
        collection.add("A");
        collection.add("B");
        collection.add(1);
        System.out.println(collection.size());
        System.out.println(collection);

        //删除
       /* collection.remove("A");
        System.out.println(collection);
        collection.clear();
        System.out.println(collection.size());*/

        //遍历
        //增强for循环
        System.out.println("==========================");
        for (Object o : collection) {
            System.out.println(o);
        }
        //迭代器，专门用来遍历集合
        System.out.println("==========================");
        Iterator it = collection.iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
            //在迭代过程中，不允许使用Collection中的方法删除，会出现并发异常
            //collection.remove(it.next());出现异常
            //it.remove();//可使用迭代器的方法
        }

        //判断
        System.out.println("===========================");
        System.out.println(collection.contains("C"));
        System.out.println(collection.isEmpty());
    }
}
```

# List子接口

特点：有序，有下标，元素可重复 

常用方法：

```java
void add(int index, E element)//在列表的指定位置插入指定元素
void clear() //从列表中移除所有元素
boolean contains(Object o)//如果列表包含指定的元素，则返回 true 
E get(int index)//返回列表中指定位置的元素。 
int indexOf(Object o)//返回此列表中第一次出现的指定元素的索引 
Iterator<E> iterator() //返回此列表的迭代器
ListIterator<E> listIterator()//返回此列表元素的列表迭代器 
E remove(int index)//移除列表中指定位置的元素
Object[] toArray(T[] a)//将此集合转化为数组a，数组a的长度若小于array.size,则自动扩容至相等，若a的程度大于array.size，则多于的元素值为null
E set(int index, E element)//用指定元素替换列表中指定位置的元素 
```

遍历时，List同样可使用增强for和迭代器 遍历，而由于List有下标，因此可用for循环来进行遍历操作

列表迭代器ListIterator，列表迭代器，允许程序员按任一方向遍历列表、迭代期间修改列表，并获得迭代器在列表中的当前位置

```java
void add(E e)//将指定的元素插入列表 
boolean hasNext()  //正向遍历，若有下一个，则返回true
boolean hasPrevious()  //逆向遍历，若有上一个，则返回true
E next()//返回列表中的下一个元素。 
int nextIndex() //返回下一个元素的索引
E previous()//返回列表中的前一个元素 
int previousIndex()  //返回上一个元素的索引
void remove()  //删除元素
void set(E e) //替换元素，可在循环中使用 List 的 set 方法替换元素
List<E> subList(int fromIndex, int toIndex)//返回一个子集合，含头不含尾
```

List的遍历方式有四种

1. 传统for循环
2. 增强型for循环
3. Collection的迭代器Iterator
4. 列表迭代器ListIterator

```java
package com.sgkurisu.collection;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.ListIterator;

//List测试
public class TestList {
    public static void main(String[] args) {
        List list = new ArrayList();
        //添加
        list.add("A");
        list.add("B");
        list.add(0, 1);
        System.out.println(list.toString());
        //删除
       /* list.remove(1);
        list.remove("B");
        System.out.println(list);*/
        //遍历
        //使用for循环
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
        //增强for循环
        for (Object object : list) {
            System.out.println(object);
        }
        //迭代器
        Iterator it = list.iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }
        //列表迭代器
        System.out.println("============使用ListIterator从前往后==============");
        ListIterator lit = list.listIterator();
        while (lit.hasNext()) {
            System.out.println(lit.nextIndex() + ":" + lit.next());
        }
        //此时，ListIterator的光标已经指向最后一个元素
        System.out.println("============使用ListIterator从后往前==============");
        while (lit.hasPrevious()) {
            System.out.println(lit.previousIndex() + ":" + lit.previous());
        }
        //获取位置
        System.out.println("A的位置:" + list.indexOf("A"));
        //获取子集合
        List list2 = list.subList(0, 2);
        System.out.println(list2.toString());
        
        //对数字1进行删除
        list.remove((Object) 1);
        System.out.println(list);

    }
}
```

集合中对数字进行操作时，list.add(20)，20为基本数据类型，在添加过程中隐含进行了装箱操作；当使用list.remove根据list中的元素进行删除时，应使用

```java
list.remove((Object) 20)
```

将20强制转换为对象

## ArrayList

ArrayList实现了List接口，除了List方法外，还有

```java
void trimToSize() //将此 ArrayList 实例的容量调整为列表的当前大小
E get(int index) //指定位置上的元素 
```

- 数组结构，查询快，增删慢
- 效率高，线程不安全

测试：

Student.java

```java
package com.sgkurisu.collection;

import java.util.Objects;

public class Student {
    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return age == student.age && Objects.equals(name, student.name);
    }
}
```

```java
package com.sgkurisu.collection;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.ListIterator;

public class TestArrayList {
    public static void main(String[] args) {
        ArrayList arrayList = new ArrayList();

        arrayList.add(new Student("A", 11));
        arrayList.add(new Student("B", 12));
        arrayList.add(new Student("C", 13));
        System.out.println(arrayList.toString());

        /*arrayList.remove(new Student("A", 11)); //根据地址删除，重写Student中的equals方法，可使用这种方法删除
        System.out.println(arrayList.toString());*/

        //遍历
        //1、传统for循环
        //2、增强for循环
        //3、迭代器
        //4、列表迭代器

        //迭代器
        System.out.println("==========迭代器============");
        Iterator it = arrayList.iterator();
        while (it.hasNext()) {
            System.out.println(it.next().toString());
        }
        //列表迭代器
        System.out.println("==========迭代器============");
        ListIterator lit = arrayList.listIterator();
        while (lit.hasNext()) {
            System.out.println(lit.next().toString());
        }
    }
}
```

和List相同，在对ArrayList进行遍历过程中不可删除或更改元素。**集合中的remove方法，是根据地址进行删除**，通过对Student中equals方法进行重写，可改变为根据name和age进行比较，然后remove

**ArrayList源码分析**：

DEFAULT_CAPACITY = 10;默认容量

elementData;为存放元素的数组

构造方法

```java
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

**当创建一个集合，未添加元素时，size容量为0；添加任意元素后，容量size为10；当集合容量满了后，每次扩容为原来的1.5倍** 

add()源码分析

```java
public boolean add(E e) {
    //size为目前的容量大小，调用ensureCapacityInternal方法
    ensureCapacityInternal(size + 1);  // Increments modCount!!  size容量+1
    elementData[size++] = e;
    return true;
}
//先调用calculateCapacity方法
private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    	//DEFAULTCAPACITY_EMPTY_ELEMENTDATA为空数组，可发现，第一次调用后，ArrayList容量为10
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
}
private void ensureExplicitCapacity(int minCapacity) {
        modCount++; //为修改次数

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity); //调用grow方法
}
//add()方法的核心
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);//oldCapacity + oldCapacity >> 1根据这个可发现，每次扩容，容量增加1.5倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
}
```

## Vector

- 数组结构，查询快，增删慢
- 效率低，线程安全

除了List接口中的类外，Vector还有

```java
Enumeration<E> elements() //返回此向量的组件的枚举
void trimToSize()// 对此向量的容量进行微调，使其等于向量的当前大小。
```

Enumeration接口中的方法，和迭代器Iterator类似

```java
boolean hasMoreElements() //测试此枚举是否包含更多的元素。 
E nextElement()//如果此枚举对象至少还有一个可提供的元素，则返回此枚举的下一个元素 
E get(int index)//指定位置的元素 
```

Vector的遍历有

1. 传统for循环
2. 增强型for循环
3. 迭代器
4. 枚举器

```java
package com.sgkurisu.collection;

import java.util.Enumeration;
import java.util.Iterator;
import java.util.Vector;

public class TestVector {
    public static void main(String[] args) {
        Vector vector = new Vector();

        vector.add("A");
        vector.add("B");
        vector.add(1);
        //删除
        /*vector.remove((Object) 1);
        vector.remove("A")
        System.out.println(vector.toString());*/
        //遍历
        for (int i = 0; i < vector.size(); i++) {
            System.out.println(vector.get(i));
        }
        for (Object object : vector) {
            System.out.println(object);
        }
        //迭代器
        Iterator it = vector.iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }
        //枚举器
        System.out.println("==============枚举器================");
        Enumeration enumeration = vector.elements();
        while (enumeration.hasMoreElements()) {
            System.out.println(enumeration.nextElement());
        }
    }
}
```

## LinkedList

- 链表结构，查询慢，增删快

常用方法：

```java
boolean add(E e)//将指定元素添加到此列表的结尾 
void add(int index, E element)//在此列表中指定的位置插入指定的元素 
void clear()//从此列表中移除所有元素 
E get(int index)//返回此列表中指定位置处的元素 
int indexOf(Object o)//返回此列表中首次出现的指定元素的索引，如果此列表中不包含该元素，则返回 -1 
int lastIndexOf(Object o)//返回此列表中最后出现的指定元素的索引，若此列表中不包含该元素，则返回 -1 
int size()//返回此列表的元素数 
Object[] toArray()//返回以适当顺序（从第一个元素到最后一个元素）包含此列表中所有元素的数组 
boolean remove(Object o) //从此列表中移除首次出现的指定元素
E remove(int index)//移除此列表中指定位置处的元素,默认删除指定位置，若想要删除目标为数字，则需remove((Object) 1)
boolean contains(Object o)//如果此列表包含指定元素，则返回 true 
```

LinkedList的遍历方式有

- 传统for循环
- 增强for循环
- Collection的迭代器Iterator
- List的列表迭代器ListIterator

**LinkedList源码分析：**

```java
transient int size = 0; //大小
transient Node<E> first;//头
transient Node<E> last;//尾
//add方法
public boolean add(E e) {
        linkLast(e);
        return true;
}
void linkLast(E e) {
        final Node<E> l = last; 
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;//第一次last为空
        else
            l.next = newNode;//之后last不为空
        size++;
        modCount++;
}
private static class Node<E> {
        E item; //存储具体内容
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
}
```

LinkedList 中创建一个私有类，用来保存要存储的数据。数组结构的ArrayList使用时开辟连续空间，而LinkedList使用时无需开辟连续空间

# 泛型

**泛型的本质是参数化类型，即把类型作为参数传递**

泛型的类型擦除见 https://www.cnblogs.com/wuqinglong/p/9456193.html

常见有泛型类、泛型接口、泛型方法、泛型集合

语法：<T,...>   T表示一种引用类型，可写多个，用逗号隔开

**常用的通配符为： T，E，K，V，？**

- ？ 表示不确定的 java 类型
- T (type) 表示具体的一个 java 类型
- K V (key value) 分别代表 java 键值中的 Key Value
- E (element) 代表 Element

优点：

- 提高代码重用性
- 防止转换异常，提高代码安全性

## 泛型类

```java
package com.sgkurisu.generic;

//泛型类
import com.oop.demo9.Test;

public class GenericClass {
    public static void main(String[] args) {
        TestGeneric<String> tg = new TestGeneric<String>(); //T只能为引用类型
        tg.t = "ABC";
        tg.test("EDF");
        System.out.println(tg.rtest());
    }
}

class TestGeneric <T> {
    T t; //不能实例化

    //方法参数
    public void test (T t) {
        System.out.println(t);
    }
    //作为方法返回值
    public T rtest () {
        return t;
    }
}
```

## 泛型接口

```java
package com.sgkurisu.generic;

//泛型接口
public interface GenericInterface<T> {
    String test = "ABC";
    T serve (T t);
}
```

泛型接口的实现方式有两种：

```java
package com.sgkurisu.generic;

//方法1，在实现方法的接口后面加上<String>
public class GenericInterfaceImpl implements GenericInterface<String> { 
    @Override
    public String serve(String s) {
        return s;
    }
}
```

```java
package com.sgkurisu.generic;

public class GenericInterfaceImpl2<T> implements GenericInterface<T> { //方法2，在类名后加<T>，这时该类变为泛型类
    @Override
    public T serve(T t) {
        return t;
    }
}
```

## 泛型方法

语法：在方法返回类型前加<T>，**作用域仅在该方法内** 

```java
package com.sgkurisu.generic;

//泛型方法
public class GenericMethod {
    public static void main(String[] args) {
        new GenericMethod().test("ABCD");//泛型参数根据传递参数决定
        new GenericMethod().test(123);
    }

    public <T> void test (T t) {
        System.out.println(t);
    }
}
```

## 泛型集合

- ArrayList
- Vector
- LinkedList
- HashSet
- TreeSet

接下来以ArrayList为例

```java
ArrayList arrayList = new ArrayList();
```

当使用不加泛型的ArrayList时，默认转换为Object，当加入add的内容较多时，输出可能不知道返回值的类型，因此强制转换输出可能会出现异常。

```java
package com.sgkurisu.generic;

import java.util.ArrayList;
import java.util.Iterator;

public class Demo {
    public static void main(String[] args) {
        ArrayList<String> arrayList = new ArrayList<String>();
        arrayList.add("ABC");
        arrayList.add("EDF");
        //arrayList.add(123);//错误，不能加非String类型数据
        for (String s : arrayList) {
            System.out.println(s);
        }
        Iterator<String> iterator = arrayList.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

当泛型对象不同时，不能转换

# Set子接口

Set中的方法均继承自Collection中的方法，Set无序、无下标、**元素不能重复**

Set接口的实现类有 HashSet, TreeSet

Set没有下标，因此不能使用传统for循环遍历，遍历方式为

1. 增强 for
2. 迭代器 Iterator

```java
package com.sgkurisu.set;

import java.util.HashSet;
import java.util.Iterator;
import java.util.Set;

public class TestSet {
    public static void main(String[] args) {
        Set<String> set = new HashSet<String>();

        //添加数据
        set.add("ABC");
        set.add("EFD");
        set.add("SQW");
        set.add("ABC");
        System.out.println(set.size());
        System.out.println(set); //输出无序
        //删除
        /*set.remove("ABC");
        System.out.println(set);
        set.clear();
        System.out.println(set);*/
        //遍历
        //增强for
        for (String s : set) {
            System.out.println(s);
        }
        //迭代器
        Iterator<String> iterator = set.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

## HashSet

- 根据元素的HashCode计算元素存放位置
- 当元素的HashCode不同时，存入；否则拒绝存入

通过分析源码，**HashSet用的为HashMap**

存储结构：哈希表（数组+链表+红黑树）

![](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202210142008035.jpeg)

遍历方式：

- 增强for
- 迭代器Iterator

**HashSet的存储过程：**

1. 根据**hashCode**计算保存位置，若为空，则直接保存，否则执行第二步
2. 执行**equals方法**，若equals为true，则重复，不保存，否则形成链表

```java
package com.sgkurisu.set;

import java.util.HashSet;
import java.util.Iterator;
import java.util.Objects;

public class TestHashSet {
    public static void main(String[] args) {
        HashSet<Student> hashSet = new HashSet<Student>();
        //添加
        hashSet.add(new Student("ABC", 11));
        hashSet.add(new Student("DEF", 11));
        hashSet.add(new Student("DFG", 11));
        System.out.println(hashSet.size());
        System.out.println(hashSet);
        //重写hashCode，未重写equals时下面代码将相同的name和age内容添加
        //重写hashCode和equals后，由于equals返回true，因此不会添加
        hashSet.add(new Student("ABC", 11));
        System.out.println(hashSet.size());

        //删除
        //重写hashCode和equals后，new后的对象hashCode和equals均为相同，因此可以删除
        hashSet.remove(new Student("ABC", 11));
        System.out.println(hashSet.size());
        //遍历
        for (Student s : hashSet) {
            System.out.println(s);
        }
        Iterator<Student> iterator = hashSet.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}

class Student {
    private String name;
    private int age;

    public Student (String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
    //重写hashCode和equals
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return age == student.age && Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        return this.name.hashCode() + this.age;
    }
}
```

## TreeSet

通过分析源码

```java
public TreeSet() {
        this(new TreeMap<E,Object>());
}
```

**TreeSet使用的为TreeMap**

特点：

- 元素不重复
- **实现了SortedSet接口，对元素自动排序**
- **元素对象类型必须实现Comparable接口，实现compareTo方法，指定排序规则**
- 通过compareTo判断是否重复，compareTo的返回值为0，则认为是重复的

遍历方式：

- 增强for
- 迭代器Iterator

存储结构：红黑树

设定比较规则的两种方法：

- 对于要添加的类实现Comparable接口中的 compareTo 方法，指定比较规则
- 创建TreeMap时使用匿名内部类实现Comparator接口，指定比较规则

```java
package com.sgkurisu.set;

import java.util.TreeSet;

public class TestTreeSet {
    public static void main(String[] args) {
        TreeSet<Person> people = new TreeSet<>();
        people.add(new Person("ABC", 11));
        people.add(new Person("C", 11));
        people.add(new Person("BC", 11));
        System.out.println(people.size());
        System.out.println(people);
    }
}

class Person {
    private String name;
    private int age;

    public Person (String name, int age) {
        this.name = name;
        this.age = age;
    }

}
```

Person类没有实现Comparable类，出现异常

![image-20221014200944769](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202210142009892.png)

对以上代码中的Person类进行修改

```java
package com.sgkurisu.set;

import java.util.TreeSet;

public class TestTreeSet {
    public static void main(String[] args) {
        TreeSet<Person> people = new TreeSet<>();
        people.add(new Person("ABC", 11));
        people.add(new Person("C", 12));
        people.add(new Person("BC", 13));
        System.out.println(people.size());
        System.out.println(people.toString());

        //删除
        people.remove(new Person("C", 12));
        System.out.println(people.size());
    }
}

class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person (String name, int age) {
        this.name = name;
        this.age = age;
    }

    //compareTo为0时，重复
    //先判断名字是否相同，若相同，则返回0；否则返回年龄差值
    @Override
    public int compareTo(Person o) {
        int n1 = this.name.compareTo(o.name); //此处的compareTo为String类的compareTo
        int n2 = this.age - o.age;
        return n1 == 0 ? n1:n2;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

Comparable接口中的compareTo类：

```java
public int compareTo(E o)//如果该对象（ti）小于、等于或大于指定对象，则分别返回负整数、零或正整数。 
```

remove方法也可进行删除

![](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202210142009175.jpeg)

> 以上为第一种方法，接下来介绍第二种方法

使用**Comparator接口**在创建TreeSet时就**指定比较规则**，实现接口中的compare类

```java
int compare(T o1, T o2)//返回负整数、零或正整数分别表示第一个参数小于、等于或大于第二个参数分
```

```java
package com.sgkurisu.set;

import java.util.Comparator;
import java.util.TreeSet;

public class TestTreeSet2 {
    public static void main(String[] args) {
        TreeSet<Person2> treeSet = new TreeSet<Person2>(new Comparator<Person2>() {
            //在创建TreeSet过程中实现Comparator接口
            @Override
            public int compare(Person2 o1, Person2 o2) {
                int n1 = o1.getName().compareTo(o2.getName()); //此处的compareTo为String类的compareTo
                int n2 = o1.getAge() - o2.getAge();
                return n1 == 0 ? n1:n2;
            }
        });
        treeSet.add(new Person2("AB", 12));
        treeSet.add(new Person2("ABC", 11));
        treeSet.add(new Person2("A", 13));
        System.out.println(treeSet.size());
        System.out.println(treeSet.toString());
    }
}

class Person2 {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Person2 (String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person2{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

# Map接口

![](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202210142009059.jpeg)

特点：

1. 存储任意键值对（Key-Value）
2. 键Key：无序、无下标、**不重复**且唯一
3. 值Value：无序、无下标、允许重复
4. 当再次添加相同的键时，先前的值将会覆盖

常用方法：

```java
V put(K key, V value) //存
V remove(Object key) //根据键删除映射
V get(Object key) //根据键返回值
int size()  //返回映射个数
void clear()  //清空
Set<Map.Entry<K,V>> entrySet()  //返回此映射中包含的映射关系的 Set 
Set<K> keySet()// 返回此映射中包含的键的 Set 
```

Map.Entry接口的方法，为键-值对的Set

```java
K getKey()//返回与此项对应的键。 
V getValue() //返回与此项对应的值 
```

遍历方法：

1. keySet()
2. entrySet()方法，返回键值对Set集合，效率更高

```java
package com.sgkurisu.map;

import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class Demo1 {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        map.put("ABC", "abc");
        map.put("DEF", "def");
        map.put("CVB", "cvb");
        System.out.println(map.toString());
        System.out.println(map.size());

        //删除
        //map.remove("ABC");

        //遍历
        //使用keySet方法
        //Set<String> set = map.keySet();
        for (String s : map.keySet()) {
            System.out.println(s + "----" + map.get(s));
        }
        //使用entrySet()方法
        //Set<Map.Entry<String, String>> entries = map.entrySet();
        for (Map.Entry<String, String> entry : map.entrySet()) {
            System.out.println(entry.getKey() + "-----" + entry.getValue());
        }


    }
}
```

## HashMap

执行效率高，线程不安全，**允许使用null作为key或value**

线程不安全指的是在并发条件下，由于链表的原因，可能会导致出现环状链表，在 get 时会出现死循环。详见 https://cloud.tencent.com/developer/article/1631902 主要是因为扩容时头插法的问题

构造方法：

```java
HashMap()//默认初始容量为16，当容量超过容量的0.75后将会进行扩容
```

存储结构：哈希树（数组+链表+红黑树），储存和HashSet相同，根据hashCode和equals判断是否重复

```java
HashMap<Student, String> students = new HashMap<>();
students.put(new Student("AA", 11), "A");//1
students.put(new Student("BB", 11), "B");
students.put(new Student("AA", 11), "A");//2
```

第一次和第二次new出的内容hashCode不同，而HashMap是根据hashCode和equals判断是否重复，因此可以添加，出现重复的key。

通过重写student的hashMap和equals方法

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Student student = (Student) o;
    return age == student.age && Objects.equals(name, student.name);
}

@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

再次执行以上程序，由于hashCode重复，equals返回为true，因此判断出二者	key相同，第二次添加失败

本例遍历如下

```java
//遍历
for (Student student : students.keySet()) {
    System.out.println(students.get(student));
}
for (Map.Entry<Student, String> entry : students.entrySet()) {
    System.out.println(entry.getKey() + "----" + entry.getValue());
}
```

```java
System.out.println(students.containsKey(new Student("AA", 11))); //返回为true
```

**源码分析**：

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16，初始容量为16
static final int TREEIFY_THRESHOLD = 8;//链表长度大于8，数组长度大于64时，改为红黑树，提高效率
static final int MAXIMUM_CAPACITY = 1 << 30;//HashMap数组最大容量
transient Node<K,V>[] table;
transient int size;//元素个数
//构造方法.刚创建HashMap未添加任何元素时，table=null，size=0，节省容量空间
public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
//put方法
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
//将会调用resize方法，该方法有一行代码为
newCap = DEFAULT_INITIAL_CAPACITY;//数组长度变为16
//当长度大于原长度的0.75时，将会扩容，每次扩容长度变为原来2倍
if (++size > threshold)
      resize();
//jdk1.8之前，链表头插入，jdk1.8之后，链表尾插入
```

## HashTable

从JDK1.0开始，线程安全，运行效率较慢，不允许使用null作为key或value

## Properties

为HashTable的子类，要求key和value均为String，通常对配置文件读取，用于流操作

## TreeMap

存储结构：红黑树

因此自定义的类作为key时，需要指定比较规则，两种方法

- 创建类时实现Comparable接口中的compareTo类
- 创建TreeMap时使用匿名内部类实现Comparator接口，指定比较规则

```java
package com.sgkurisu.map;

import java.util.Comparator;
import java.util.Map;
import java.util.TreeMap;

public class TestTreeMap {
    public static void main(String[] args) {
        TreeMap<Person, String> treeMap = new TreeMap<>(new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                int n1 = o1.getName().compareTo(o2.getName());
                int n2 = o1.getAge() - o2.getAge();
                return n1 == 0 ? n1:n2;
            }
        });
        treeMap.put(new Person("A", 11), "AAA");
        treeMap.put(new Person("B", 11), "BBB"); //compareTo判断名字不同，因此返回年龄差值，为0
        treeMap.put(new Person("A", 11), "AAA"); //compareTo判断为0，重复
        treeMap.put(new Person("B", 12), "BBB"); //和A相比返回-1，因此判断B>A
        System.out.println(treeMap.size());
        System.out.println(treeMap.toString());
        //遍历
        for (Person person : treeMap.keySet()) {
            System.out.println(treeMap.get(person));
        }
        for (Map.Entry<Person, String> entry : treeMap.entrySet()) {
            System.out.println(entry.getKey().getName() + "----------" + entry.getValue());
        }
    }
}

class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public int compareTo(Person o) {
        int n1 = this.name.compareTo(o.name);
        int n2 = this.age - o.age;
        return n1 == 0 ? n1:n2;
    }
}
```

# Collections工具类

```java
static <T> ArrayList<T>  list(Enumeration<T> e)// 从小到大排序
static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) //在list已排序的前提下二分查找相应的key
static <T> void copy(List<? super T> dest, List<? extends T> src)//src集合复制到dest集合中，要求二者size必须相同
static void reverse(List<?> list)//反转list集合中的元素顺序
static void shuffle(List<?> list)//将list中元素顺序打乱
```

数组和集合之间的转换：

- 集合转为数组：

  ```java
  <T> T[] toArray(T[] a) //将集合转换为数组并返回数组
  ```

- 数组转为集合：

  ```java
  Arrays.asList(T... a) //将数组a转换为集合，并返回List类型。
      				  //数组转换为集合后，该集合为受限集合，不能添加或删除
  ```

基本数据类型转换为集合时，需要先转换为包装类

```java
package com.sgkurisu.collections;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

//collections工具类的使用
public class TestCollections {
    public static void main(String[] args) {
        ArrayList<Integer> arrayList = new ArrayList<>();
        arrayList.add(10);
        arrayList.add(3);
        arrayList.add(5);
        arrayList.add(7);
        arrayList.add(2);
        System.out.println(arrayList);
        //排序
        Collections.sort(arrayList);
        System.out.println(arrayList);
        //二分法查找
        System.out.println(Collections.binarySearch(arrayList,2));
        //逆序
        Collections.reverse(arrayList);
        System.out.println(arrayList);
        //打乱
        Collections.shuffle(arrayList);
        System.out.println(arrayList);
        //转为数组
        Integer[] integer = arrayList.toArray(new Integer[0]);
        for (Integer temp : integer) {
            System.out.println(temp);
        }
        //数组转为集合
        List<Integer> list = Arrays.asList(integer);
        System.out.println(list.toString());
    }
}
```
