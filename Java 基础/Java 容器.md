# 1、概览

容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对映射表。

## Collection

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209221716625.png)

### 1、Set 接口

特点：无序、无下标、**元素不能重复** 

Set没有下标，因此不能使用传统for循环遍历，遍历方式为增强 for 循环和迭代器 Iterator

- **TreeSet**：基于红黑树实现，**TreeSet使用的为TreeMap** 。

  特点：

  - 元素不重复
  - 实现了SortedSet接口，对元素自动排序
  - **元素对象类型必须实现Comparable接口，实现compareTo方法，指定排序规则** ；也可以在创建 TreeSet 时就指定比较规则，实现 **Comparator接口** 中的 compare 类。
  - 通过元素对象中的 compareTo 方法判断是否重复，compareTo 的返回值为 0，则认为是重复的

  查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。

- **HashSet** ：基于哈希表实现， **HashSet用的为HashMap** 。

  - 根据 hashCode 计算保存位置，若为空，则直接保存
  - 若 hashCode 相同，则执行 equals 方法，若 equals 为 true，则重复，不保存，否则形成链表

  不支持有序性操作，遍历方式有增强 for 和迭代器 Iterator，使用 Iterator 遍历 HashSet 得到的结果是不确定的

- **LinkedHashSet**：具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。

### 2、List 接口

特点：有序，有下标，元素可重复。遍历方式：传统 for 循环，增强 for 循环，Collection 的迭代器 Iterator，列表迭代器 ListIterator

- **ArrayList**：基于动态数组实现，支持随机访问。

  特点：

  - 数组结构，查询快，增删慢
  - 效率高，线程不安全

- **Vector**：和 ArrayList 类似，不过 Vector 是线程安全的

  特点：

  - 数组结构，查询快，增删慢
  - 效率低，线程安全

- **LinkedList**：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。

  特点：链表结构，查询慢，增删快

### 3、Queue 接口

- **LinkedList**：实现双向队列。

- **PriorityQueue**：完全二叉树中的小顶堆（任意一个根节点的权值，都不大于其左右子节点的权值）

  ![PriorityQueue_base.png](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209221716982.png)

  父子节点具有如下关系：

  ```html
  leftNo = parentNo * 2 + 1
  rightNo = parentNo * 2 + 2
  parentNo = (nodeNo - 1) / 2
  ```

## Map

特点：

1. 存储任意键值对（Key-Value）
2. 键Key：无序、无下标、**不重复**且唯一
3. 值Value：无序、无下标、允许重复
4. 当再次添加相同的键时，先前的值将会覆盖

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209221716844.png)

- **HashMap**：基于哈希表实现，储存和 HashSet 相同，根据 hashCode 和 equals 判断是否重复

  特点：执行效率高，线程不安全，允许使用 null 作为 key 或 value

- **TreeMap**：基于红黑树实现，和 TreeSet 相同，TreeSet 使用的为 TreeMap
  
  - 待添加的对象实现 Comparable 中的 compareTo 类
  - 创建 TreeMap 时使用匿名内部类实现 Comparator 接口，指定比较规则
  
- **HashTable**：和 HashMap 类似，但它是线程安全的，保证线程安全的方法是在 HashTable 类中的每个方法前加入 synchronized，这意味着同一时刻多个线程同时写入 HashTable 不会导致数据不一致，这也导致了运行效率较慢，不允许使用 null 作为 key 或 value。HashTable 是遗留类，不应该去使用它，而是使用 ConcurrentHashMap 来支持线程安全，ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。

- **LinkedHashMap**：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

# 2、容器中的设计模式

## 迭代器模式

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209221716571.png)

Collection 继承了 Iterable 接口，其中的 iterator() 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。

Iterator 和 Iterable 区别：

Iterator 和 Iterable 都是两个接口，Iterator 从 JDK 1.2 开始使用，Iterable 从 JDK 1.5 开始使用，源码如下：

Iterator 中只有两个方法： hasNext 和 next，分别用来判断是否还有下个元素，和返回下个元素

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    //两个 default 方法
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

Iterable 中只有一个成员 Iterator，因此用 Iterable 可产生 Iterator

```java
public interface Iterable<T> {
    Iterator<T> iterator();
    //两个 default 方法
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

从 JDK 1.5 之后可以使用 foreach 方法来遍历实现了 Iterable 接口的聚合对象。也就是增强 for 循环来进行访问

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
for (String item : list) {
    System.out.println(item);
}
```

## 适配器模式

java.util.Arrays.asList() 方法可以把数组类型转换为 List 类型，并且转换后的 List 大小不可变，不能增加或删除元素。注意：使用 asList 时，不能使用基本类型数组作为参数，因为asList 接受的参数是一个泛型的变长参数，而基本数据类型无法泛型转化，因此 8 个基本的数据类型不可作为 asList 的参数，只能使用相应的包装类型数组。

```java
Integer[] arr = {1, 2, 3};
List list = Arrays.asList(arr);
```

也可以使用以下方式调用 asList()：

```java
List list = Arrays.asList(1, 2, 3);
```

# 3、源码分析

## ArrayList

### 1、概览

因为 ArrayList 是基于数组实现的，所以支持快速随机访问。RandomAccess 接口标识着该类支持快速随机访问。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

当创建 ArrayList 后，未添加元素时，容量为 0；添加任意元素后，容量为 10，数组默认大小为 10

```java
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

```java
private static final int DEFAULT_CAPACITY = 10;
```

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209221717600.png)

### 2、扩容

添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，即 oldCapacity+oldCapacity/2。其中 oldCapacity >> 1 需要取整，所以新容量大约是旧容量的 1.5 倍左右。（oldCapacity 为偶数就是 1.5 倍，为奇数就是 1.5 倍-0.5）

扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

```java
//构造方法，指定初始容量
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
}
```

### 3、删除元素

需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)，可以看到 ArrayList 删除元素的代价是非常高的。

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

### 4、序列化

ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。

保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。

```java
transient Object[] elementData; // non-private to simplify nested class access
```

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

序列化是调用ObjectOutputStream 的 writeObject() 将对象转换为字节流并输出。而 writeObject() 方法在传入的对象存在 writeObject() 的时候会去反射调用该对象的 writeObject() 来实现序列化。

反序列化调用的是 ObjectInputStream 的 readObject() 方法，也将会调用对象的 readObject() 方法。

步骤：

a) 写入

- 首先创建一个OutputStream输出流；
- 然后创建一个ObjectOutputStream输出流，并传入OutputStream输出流对象；
- 最后调用ObjectOutputStream对象的writeObject()方法将对象状态信息写入OutputStream。

b)读取

- 首先创建一个InputStream输入流；
- 然后创建一个ObjectInputStream输入流，并传入InputStream输入流对象；
- 最后调用ObjectInputStream对象的readObject()方法从InputStream中读取对象状态信息。

```java
ArrayList list = new ArrayList();
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
oos.writeObject(list);
```

### 5、Fail-Fast

modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。

## Vector

### 1、同步

Vector 是线程安全的，不过实现线程安全的方法是在每个方法上使用 synchronized 进行同步。

```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
```

### 2、扩容

Vector 的构造函数可指定初始容量（若未指定初始容量，则默认长度为 10），此时capacityIncrement 值被设置为 0。Vector 的构造函数还可以传入 capacityIncrement 参数，它的作用是在扩容时使容量 capacity 增长 capacityIncrement。如果这个参数的值小于等于 0，扩容时每次都令 capacity 为原来的两倍。也就是说默认情况下 Vector 每次扩容时容量都会翻倍。

```java
public Vector(int initialCapacity) {
    this(initialCapacity, 0);
}

public Vector() {
    this(10);
}
```

```java
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
```

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### 3、与 ArrayList 的比较

- Vector 由于使用了 synchronized 进行同步，因此线程安全，但同时相比于 ArrayList 效率更低。ArrayList 线程不安全，效率更高，同步操作需要由程序员自己控制。
- Vector 默认情况下每次扩容为初始容量的 2 倍（也可手动指定 capacityIncrement），ArrayList 每次扩容为初始容量的 1.5 倍。

### 4、替代方案

1. 使用 `Collections.synchronizedList();` 得到一个线程安全的 ArrayList。

   ```java
   List<String> list = new ArrayList<>();
   List<String> synList = Collections.synchronizedList(list);
   ```

2. 使用 concurrent 并发包下的 CopyOnWriteArrayList 类。

   ```java
   List<String> list = new CopyOnWriteArrayList<>();
   ```

## CopyOnWriteArrayList

### 1、读写分离

- 写操作在一个**复制的数组**上进行，读操作还是在**原始数组**中进行，读写分离，互不影响。

- **写操作需要加锁**，防止并发写入时导致写入数据丢失。
- 写操作结束之后需要把**原始数组指向新的复制数组**。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

final void setArray(Object[] a) {
    array = a;
}
```

```java
@SuppressWarnings("unchecked")
private E get(Object[] a, int index) {
    return (E) a[index];
}
```

### 2、适用场景

CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

缺陷：

- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。

所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。

## LinkedList

### 1、概览

基于**双向链表**实现，使用 Node 存储链表节点信息。

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```

每个链表存储了 first 和 last 指针，分别指向表头和表尾。

```java
transient Node<E> first;
transient Node<E> last;
```

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/20210527110546.png)

### 2、与 ArrayList 的比较

ArrayList 基于动态数组实现，LinkedList 基于双向链表实现。ArrayList 和 LinkedList 的区别可以归结为数组和链表的区别：

- 数组支持随机访问，但插入删除的代价很高，需要移动大量元素（读代价低，写代价高）
- 链表不支持随机访问，但插入删除只需要改变指针（读代价高，写代价低）

## HashMap

以下源码以 JDK 1.7  为主

### 1、存储结构

内部包含了一个 Entry 类型的数组 table，HashMap 默认大小为 16。Entry 存储着键值对。它包含了四个字段（key, value, next, hash），从 next 字段我们可以看出 **Entry 是一个链表**。数组中的每个位置被当成一个桶，一个桶存放一个链表。HashMap 使用拉链法来解决冲突，同一个链表中存放哈希值和散列桶取模运算结果相同的 Entry。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/20210527111413.png)

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

```java
transient Entry[] table;
```

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;

    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }

    public final K getKey() {
        return key;
    }

    public final V getValue() {
        return value;
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }

    public final int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    public final String toString() {
        return getKey() + "=" + getValue();
    }
}
```

### 2、拉链法的工作原理

```java
HashMap<String, String> map = new HashMap<>();
map.put("K1", "V1");
map.put("K2", "V2");
map.put("K3", "V3");
```

- 新建一个 HashMap，默认大小为 16；
- 插入 <K1,V1> 键值对，先计算 K1 的 hashCode 为 115，使用除留余数法得到所在的桶下标 115%16=3。
- 插入 <K2,V2> 键值对，先计算 K2 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6。
- 插入 <K3,V3> 键值对，先计算 K3 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6，插在 <K2,V2> 前面。

注意，链表的插入是以**头插法**方式进行的，例如上面的 <K3,V3> 不是插在 <K2,V2> 后面，而是插入在链表头部。

**查找**分两步进行：

- 计算键值对所在的桶；
- 在链表上顺序查找，时间复杂度显然和链表的长度成正比。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/20210527112054.png)

### 3、put 操作

put 源码：

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    // 键为 null 单独处理
    if (key == null)
        return putForNullKey(value);
    //获得 key 的哈希值
    int hash = hash(key);
    // 确定桶下标
    int i = indexFor(hash, table.length);
    // 先找出是否已经存在键为 key 的键值对，如果存在的话就更新这个键值对的值为 value
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    
    modCount++;
    // 插入新键值对，头插法
    addEntry(hash, key, value, i);
    return null;
}
```

HashMap 允许插入键为 null 的键值对。但是因为无法调用 null 的 hashCode() 方法，也就无法确定该键值对的桶下标，只能通过强制指定一个桶下标来存放。**HashMap 使用第 0 个桶存放键为 null 的键值对**。

```java
private V putForNullKey(V value) {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
```

使用链表的头插法，也就是新的键值对插在链表的头部，而不是链表的尾部。数组 table 的扩容每次为之前长度的2 倍。

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        //扩容，2 倍
        resize(2 * table.length);
        //判断 key 是否为空
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}

void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    // 头插法，链表头部指向新的键值对
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```

```java
Entry(int h, K k, V v, Entry<K,V> n) {
    value = v;
    next = n;
    key = k;
    hash = h;
}
```

JDK 1.8 table 长度每次扩容为原来的 2 倍，插入时当 key 的 hash 相同时插入到链表**尾部**

![image-20210527114056952](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/20210527114057.png)

### 4、确定桶下标

很多操作都需要先确定一个键值对所在的桶下标。

```java
int hash = hash(key);
int i = indexFor(hash, table.length);
```

#### 4.1 计算 hash 值

```java
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

```java
public final int hashCode() {
    return Objects.hashCode(key) ^ Objects.hashCode(value);
}
```

在 JDK 1.8 中

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

#### 4.2 取模

令 x = 1<<4，即 x 为 2 的 4 次方，它具有以下性质：

```text
x   : 00010000
x-1 : 00001111
```

令一个数 y 与 x-1 做与运算，可以去除 y 位级表示的第 4 位以上数：

```text
y       : 10110010
x-1     : 00001111
y&(x-1) : 00000010
```

这个操作和 y 对 x 取模的效果是一样的

```text
y   : 10110010
x   : 00010000
y%x : 00000010
```

位运算的代价比求模运算小的多，因此在进行这种计算时用位运算的话能带来更高的性能。

确定桶下标的最后一步是将 key 的 hash 值对桶个数取模：hash % capacity，**如果能保证 capacity 为 2 的 n 次方**，那么就可以将这个操作转换为位运算。

```java
static int indexFor(int h, int length) {
    return h & (length-1);
}
```

### 5、扩容-基本原理

设 HashMap 的 table 长度为 M，需要存储的键值对数量为 N，如果哈希函数满足**均匀性**的要求，那么每条链表的长度大约为 N/M，因此查找的复杂度为 O(N/M)。

为了让查找的成本降低，应该使 N/M 尽可能小，因此需要保证 M 尽可能大，也就是说 table 要尽可能大。HashMap 采用**动态扩容**来根据当前的 N 值来调整 M 值，使得空间效率和时间效率都能得到保证。

和扩容相关的参数主要有：capacity、size、threshold 和 load_factor。

|    参数    | 含义                                                         |
| :--------: | :----------------------------------------------------------- |
|  capacity  | table 的容量大小，默认为 16。需要注意的是 capacity 必须保证为 2 的 n 次方。因为在取模过程中用到了位与运算 |
|    size    | 键值对数量。                                                 |
| threshold  | size 的临界值，当 size 大于等于 threshold 就必须进行扩容操作。 |
| loadFactor | 装载因子，默认 = DEFAULT_LOAD_FACTOR = 0.75f，table 能够使用的比例，threshold = (int)(capacity* loadFactor)。 |

默认情况下，table 的容量为 16。当 table 中已经使用的大小超过 75% 时，将会扩容，每次扩容为原来的 2 倍。

```java
static final int DEFAULT_INITIAL_CAPACITY = 16;

static final int MAXIMUM_CAPACITY = 1 << 30;

static final float DEFAULT_LOAD_FACTOR = 0.75f;

transient Entry[] table;

transient int size;

int threshold;

final float loadFactor;

transient int modCount;
```

当需要扩容时，令 capacity 为原来的两倍。

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    if (size++ >= threshold)
        //扩容为原来的 2 倍
        resize(2 * table.length);
}
```

扩容使用 resize() 实现，需要注意的是，扩容操作同样需要把 oldTable 的所有键值对重新插入 newTable 中，因此这一步是很费时的。

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable);
    table = newTable;
    threshold = (int)(newCapacity * loadFactor);
}

void transfer(Entry[] newTable) {
    Entry[] src = table;
    int newCapacity = newTable.length;
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
}
```

### 6、扩容-重新计算桶下标

在对 table 扩容后，需要把原来的键值对重新计算桶下标，放到对应的桶上。在前面提到，HashMap 使用 hash%capacity 来确定桶下标。HashMap capacity 为 2 的 n 次方这一特点能够极大降低重新计算桶下标操作的复杂度。

假设原数组长度 capacity 为 16，扩容之后 new capacity 为 32：

```html
capacity     : 00010000
new capacity : 00100000
```

### 7、计算数组容量

HashMap 构造函数也允许用户传入的容量不是 2 的 n 次方，可自动地将传入的容量转换为 2 的 n 次方。

可通过如下方法获得一个数的掩码：对于 10010000，它的掩码为 11111111

```text
mask |= mask >> 1    11011000
mask |= mask >> 2    11111110
mask |= mask >> 4    11111111
```

而 mask + 1 是大于原始数字的最小的 2 的 n 次方。mask 为掩码

```text
num     10010000
mask+1  100000000
```

HashMap 中计算数组容量的代码：

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

通过以下源码可发现，将掩码赋值给 threshold，loadFactor 为默认值 0.75

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

### 8、链表转红黑树

从 JDK 1.8 开始，当数组长度大于 64，链表长度大于长度大于等于 8 时会将链表转换为红黑树。

### 9、与 Hashtable 的比较

- Hashtable 是线程安全的，实现线程安全是通过在每个方法前加 synchronized，因此效率较低
- Hashtable 不允许使用 null 作为 key 或 value，HashMap 允许使用 null 作为 key 或 value
- HashMap 的迭代器是 fail-fast 迭代器（在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException）
- HashMap 不能保证随着时间的推移 Map 中的元素次序不变

## TreeMap

与 HashMap 相比，TreeMap 是一个能比较元素大小的 Map 集合，会对传入的 key 进行了大小排序。其中，可以使用元素的自然顺序，也可以使用集合中自定义的比较器来进行排序；TreeMap 底层是红黑树。

TreeMap 的继承结构如下：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202111251020926.webp)

TreeMap继承于AbstractMap，实现了Map, Cloneable, NavigableMap, Serializable接口。

(1)TreeMap 继承于AbstractMap，而AbstractMap实现了Map接口，并实现了Map接口中定义的方法，减少了其子类继承的复杂度；

(2)TreeMap 实现了Map接口，成为Map框架中的一员，可以包含着key--value形式的元素；

(3)TreeMap 实现了NavigableMap接口，意味着拥有了更强的元素搜索能力；

(4)TreeMap 实现了Cloneable接口，实现了clone()方法，可以被克隆；

(5)TreeMap 实现了Java.io.Serializable接口，支持序列化操作，可通过Hessian协议进行传输；

**SortedMap**

SortedMap接口中定义了一个方法 Comparator<? super K> comparator()；该方法决定了TreeMap体系的走向，有了比较器，就可以对插入的元素进行排序了；

**NavigableMap**

NavigableMap 接口又是对 SortedMap 进一步的扩展：主要增加了对集合内元素的搜索获取操作，NavigableMap 的目的很简单、很直接，就是增强了对集合内元素的搜索、获取的功能，当子类 TreeMap 实现时，自然获取以上的功能；

TreeMap具有如下特点：

- 不允许出现重复的key；
- 可以插入null键，null值；
- 可以对元素进行排序；
- 无序集合（插入和遍历顺序不一致）；

## ConcurrentHashMap

### 1、存储结构

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/20210528200902.png)

```java
static final class HashEntry<K,V> {
    final int hash;
    final K key;
    volatile V value;
    volatile HashEntry<K,V> next;
}
```

ConcurrentHashMap 和 HashMap 实现上类似，最主要的差别是 ConcurrentHashMap 采用了分段锁（Segment），**每个分段锁维护着几个桶（HashEntry）**，多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。

Segment 继承自 ReentrantLock。

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {

    private static final long serialVersionUID = 2249069246763182397L;

    static final int MAX_SCAN_RETRIES =
        Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;

    transient volatile HashEntry<K,V>[] table;

    transient int count;

    transient int modCount;

    transient int threshold;

    final float loadFactor;
}
```

```java
final Segment<K,V>[] segments;
```

默认的并发级别为 16，也就是说默认创建 16 个 Segment。

```java
static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

### 2、size 方法

每个 Segment 维护了一个 count 变量来统计该 Segment 中的键值对个数。在执行 size 操作时，需要遍历所有 Segment 然后把 count 累计起来。

ConcurrentHashMap 在执行 size 操作时先尝试不加锁，如果连续两次不加锁操作得到的结果一致，那么可以认为这个结果是正确的。

总共尝试的次数为 3 次，如果尝试的次数超过 3 次，就需要对每个 Segment 加锁。

```java
static final int RETRIES_BEFORE_LOCK = 2;

public int size() {
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum;         // sum of modCounts
    long last = 0L;   // previous sum
    int retries = -1; // first iteration isn't retry
    try {
        for (;;) {
            // 超过尝试次数，则对每个 Segment 加锁
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            // 连续两次得到的结果一致，则认为这个结果是正确的
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE : size;
}
```

### 3、JDK 1.8 的改动

JDK 1.7 使用分段锁机制来实现并发更新操作，核心类为 Segment，它继承自重入锁 ReentrantLock，并发度与 Segment 数量相等。

JDK 1.8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized。

并且 JDK 1.8 的实现也在链表过长时会转换为红黑树。

## LinkedHashMap

### 1、存储结构

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209222016533.jpeg)

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

内部节点 Node 继承自 HashMap 的 Node，底层仍使用的是数组 + 链表的方式，通过额外添加 head 和 tail 节点，维护双向链表。

LinkedHashMap 继承自 HashMap，因此具有和 HashMap 一样的快速查找特性。

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
```

根据链表中元素的顺序可以将 LinkedHashMap 分为：**保持插入顺序的 LinkedHashMap** 和 **保持访问顺序的LinkedHashMap** 。LinkedHashMap 内部维护了一个双向链表，用来维护插入顺序或者访问顺序。

```java
transient LinkedHashMap.Entry<K,V> head;
transient LinkedHashMap.Entry<K,V> tail;
```

accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序。

```java
final boolean accessOrder;
```

LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用。

```java
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
```

### 2、afterNodeAccess()

如果 **accessOrder 为 true**，那么指定为访问顺序，也即是 LRU 规则，在每次访问一个节点时，**调用 afterNodeAccess，将这个节点移到链表尾部**，保证链表**尾部**是最近访问的节点，那么链表首部就是最近最久未使用的节点。

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

### 3、afterNodeInsertion()

afterNodeInsertion() 方法在 put 等操作后执行，在 afterNodeInsertion() 方法中，**当 removeEldestEntry() 方法返回 true 时（即链表长度大于规定的最大长度）会移除最不常访问的节点，也就是链表首部节点 first。**

afterNodeInsertion 方法默认返回为 false，在 LRU 缓存下可返回 `size() > capacity` 判断是否当前容量大于指定容量。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

removeEldestEntry() 默认为 false，如果需要让它为 true，需要继承 LinkedHashMap 并且覆盖这个方法的实现，这在**实现 LRU 的缓存**中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据。

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

### 4、LRU 缓存

以下是使用 LinkedHashMap 实现的一个 LRU 缓存：

- 设定最大缓存空间 MAX_ENTRIES 为 3；
- 使用 LinkedHashMap 的构造函数将 accessOrder 设置为 true，开启访问顺序；
- 覆盖 removeEldestEntry() 方法实现，在节点多于 MAX_ENTRIES 就会将最近最久未使用的数据移除。

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private static final int MAX_ENTRIES = 3;

    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }

    LRUCache() {
        super(MAX_ENTRIES, 0.75f, true);
    }
}
```

测试：

```java
public class Test {
    public static void main(String[] args) {
        LRUCache<String, Integer> cache = new LRUCache<>();
        cache.put("a", 1);
        cache.put("b", 2);
        cache.put("c", 3);
        cache.get("a");
        cache.put("d", 4);
        System.out.println(cache.keySet());
    }
}

class LRUCache<K, V> extends LinkedHashMap<K, V> {

    //最大缓存容量
    private static final int MAX_ENTRIES = 3;

    //设置为最大缓存容量，装载因子，访问顺序
    LRUCache() {
        super(MAX_ENTRIES, 0.75f, true);
    }

    //覆盖 removeEldestEntry() 方法
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > MAX_ENTRIES;
    }
}
```

输出：[c, a, d]

![image-20210528210647507](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/20210528210647.png)

> 构造方法

```java
public LinkedHashMap() {
    super();
    accessOrder = false;
}
```

```java
public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}
```

```java
public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}
```

```java
public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

**实现 LRU 缓存**是 leetcode 146 中的题目，只使用 LinkedHashMap 实现没有什么意义，如下为自己实现的 HashMap 结合双向链表实现 LRU，如下自己实现的 LRU 中，使用双向链表和 HashMap 进行维护，链表用来维护 LRU 规则，最近访问的放到链表首部（ LinkedHashMap 中最近访问的放入的是链表尾部）；HashMap 用来快速访问。

```java
public class LRUCache {
    private class Node {
        int key;
        int val;
        Node pre;
        Node next;
        Node() {}
        Node(int key, int val) {
            this.key = key;
            this.val = val;
        }
    }

    int capacity;
    int size;
    HashMap<Integer, Node> hashMap;
    Node head;
    Node tail;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        size = 0;
        hashMap = new HashMap<>();
        head = new Node();
        tail = new Node();
        head.next = tail;
        tail.pre = head;
    }

    public int get(int key) {
        Node node = hashMap.get(key);
        if (node == null) {
            return -1;
        }
        moveToHead(node);
        return node.val;
    }

    public void put(int key, int val) {
        Node node = hashMap.get(key);
        if (node == null) {
            node = new Node(key, val);
            if (size == capacity) {
                deleteTail();
            }
            hashMap.put(key, node);
            addToHead(node);
        } else {
            node.val = val;
        }
    }

    private void deleteTail() {
        int key = tail.pre.key;
        hashMap.remove(key);
        delete(tail.pre);
        size--;
    }

    private void moveToHead(Node node) {
        delete(node);
        addToHead(node);
    }

    private void addToHead(Node node) {
        node.next = head.next;
        node.pre = head;
        head.next.pre = node;
        head.next = node;
        size++;
    }

    private void delete(Node node) {
        node.pre.next = node.next;
        node.next.pre = node.pre;
        size--;
    }
}
```

## WeakHashMap

### 1、存储结构

WeakHashMap 主要用来实现缓存，通过使用 WeakHashMap 来引用缓存对象，由 JVM 对这部分缓存进行回收。

WeakHashMap 的 Entry 继承自 WeakReference，被 WeakReference 关联的对象在下一次垃圾回收时会被回收。

```java
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V>
```

### 2、ConcurrentCache

Tomcat 中的 ConcurrentCache 使用了 WeakHashMap 来实现缓存功能。

ConcurrentCache 采取的是分代缓存：

- 经常使用的对象放入 eden 中，eden 使用 ConcurrentHashMap 实现，不用担心会被回收；
- 不常用的对象放入 longterm，longterm 使用 WeakHashMap 实现，这些老对象会被垃圾收集器回收。
- 当调用 get() 方法时，会先从 eden 区获取，如果没有找到的话再到 longterm 获取，当从 longterm 获取到就把对象放入 eden 中，从而保证经常被访问的节点不容易被回收。
- 当调用 put() 方法时，如果 eden 的大小超过了 size，那么就将 eden 中的所有对象都放入 longterm 中，利用虚拟机回收掉一部分不经常使用的对象。

```java
public final class ConcurrentCache<K, V> {

    private final int size;

    private final Map<K, V> eden;

    private final Map<K, V> longterm;

    public ConcurrentCache(int size) {
        this.size = size;
        this.eden = new ConcurrentHashMap<>(size);
        this.longterm = new WeakHashMap<>(size);
    }

    public V get(K k) {
        V v = this.eden.get(k);
        if (v == null) {
            v = this.longterm.get(k);
            if (v != null)
                this.eden.put(k, v);
        }
        return v;
    }

    public void put(K k, V v) {
        if (this.eden.size() >= size) {
            this.longterm.putAll(this.eden);
            this.eden.clear();
        }
        this.eden.put(k, v);
    }
}
```
