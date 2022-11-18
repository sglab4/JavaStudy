参考 https://coolshell.cn/articles/9606.html

# 1、概述

首先我们知道，HashMap 使用数组加链表的形式存储元素，当一个key被加入时，会通过Hash算法通过key算出这个数组的下标i，然后就把这个<key, value>插到table[i]中，如果有两个不同的key被算在了同一个i，那么就叫冲突，又叫碰撞，这样会在table[i]上形成一个链表。

Hash表的尺寸和容量非常的重要。一般来说，Hash表这个容器当有数据要插入时，都会检查容量有没有超过设定的thredhold，如果超过，需要增大Hash表的尺寸，但是这样一来，整个Hash表里的无素都需要被重算一遍。这叫rehash，这个成本相当的大。

# 2、源码

```java
public V put(K key, V value)
{
    ......
    //算Hash值
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    //如果该key已被插入，则替换掉旧的value （链接操作）
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
    //该key不存在，需要增加一个结点
    addEntry(hash, key, value, i);
    return null;
}
```

```java
void addEntry(int hash, K key, V value, int bucketIndex)
{
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    //查看当前的size是否超过了我们设定的阈值threshold，如果超过，需要resize
    if (size++ >= threshold)
        resize(2 * table.length);
} 
```

新建一个更大尺寸的hash表，然后把数据从老的Hash表中迁移到新的Hash表中。

```java
void resize(int newCapacity)
{
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    ......
    //创建一个新的Hash Table
    Entry[] newTable = new Entry[newCapacity];
    //将Old Hash Table上的数据迁移到New Hash Table上
    transfer(newTable);
    table = newTable;
    threshold = (int)(newCapacity * loadFactor);
}
```

转移源数据，节点转移时，在 jdk1.7 及之前，采用的是**头插法**

```java
void transfer(Entry[] newTable)
{
    Entry[] src = table;
    int newCapacity = newTable.length;
    //下面这段代码的意思是：
    //  从OldTable里摘一个元素出来，然后放到NewTable中
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;   // 1
                int i = indexFor(e.hash, newCapacity); // 2
                e.next = newTable[i]; // 3
                newTable[i] = e; // 4
                e = next; // 5
            } while (e != null);
        }
    }
} 
```

# 3、例子

## 单线程 ReHash

- 我假设了我们的hash算法就是简单的用key mod 一下表的大小（也就是数组的长度）。

- 最上面的是old hash 表，其中的Hash表的size=2, 所以key = 3, 7, 5，在mod 2以后都冲突在table[1]这里了。

- 接下来的三个步骤是Hash表 resize成4，然后所有的<key,value> 重新rehash的过程

![image-20211130152510337](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111301525416.png)

## 并发 ReHash

假设有两个线程

```java
do {
    Entry<K,V> next = e.next; // <--假设线程一执行到这里就被调度挂起了
    int i = indexFor(e.hash, newCapacity);
    e.next = newTable[i];
    newTable[i] = e;
    e = next;
} while (e != null);
```

线程一被挂起后，线程二执行完成了。于是我们有下面的这个样子。

![image-20211130152640287](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111301526344.png)

注意，**因为Thread1的 e 指向了key(3)，而next指向了key(7)，其在线程二rehash后，指向了线程二重组后的链表**。

线程二执行完成后调度线程一

- **先是执行 newTalbe[i] = e;**
- **然后是e = next，导致了e指向了key(7)，**
- **而下一次循环的next = e.next导致了next指向了key(3)**

![image-20211130152754771](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111301527819.png)

线程一接着工作，采用头插法，**把key(7)摘下来，放到newTable[i]的第一个，然后把e和next往下移**。

![image-20211130152809791](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111301528847.png)

**e.next = newTable[i] 导致 key(3).next 指向了 key(7)**

**注意：此时的key(7).next 已经指向了key(3)， 环形链表就这样出现了。**

![image-20211130152836948](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111301528006.png)

于是，环形链表出现，当线程一使用方法 get(3) 时，将会出现死循环

概括来说，在多线程下由于线程挂起，造成了链表指向混乱。

# 4、jdk1.8 中

以上出现的环形链表出现在 jdk1.7 及之前的版本中，在 jdk1.8 中，多线程情况下可能会出现数据覆盖的问题。

