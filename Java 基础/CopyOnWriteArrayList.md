参考 https://www.jianshu.com/p/cd7a73e6bd78

# 1、概述

CopyOnWriteArrayList 是ArrayList的线程安全版本，从他的名字可以推测，CopyOnWriteArrayList是在有写操作的时候会copy一份数据，然后写完再设置成新的数据。CopyOnWriteArrayList适用于读多写少的并发场景。

继承结构如下：

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061449005.webp)

可以看到它实现了List接口，如果去看ArrayList的类图的话，可以发现也是实现了List接口，也就得出一句废话，ArrayList提供的api，CopyOnWriteArrayList也提供，下文中来分析CopyOnWriteArrayList是如何来做到线程安全的实现读写数据的，而且也会顺便对比ArrayList的等效实现为什么不支持线程安全的。下面首先展示了CopyOnWriteArrayList中比较重要的成员：

```java
/** The lock protecting all mutators */
final transient ReentrantLock lock = new ReentrantLock();

/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array;
```

CopyOnWriteArrayList使用了ReentrantLock来支持并发操作，array就是实际存放数据的数组对象。ReentrantLock是一种支持重入的独占锁，任意时刻只允许一个线程获得锁，所以可以安全的并发去写数组。

# 2、add 和 set 方法

根据源码可发现，add 方法中，首先获得原始数组对象，然后再使用 Arrays.copyOf 方法创建一个原始数组长度 + 1 的新数组，将原始数组复制到新数组中，在新数组末尾赋值需要添加的值，最后调用 setArray 方法将原始数组指向新数组。

```java
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock(); //上锁，只允许一个线程进入
        try {
            Object[] elements = getArray(); // 获得当前数组对象
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);//拷贝到一个新的数组中
            newElements[len] = e;//插入数据元素
            setArray(newElements);//将新的数组对象设置回去
            return true;
        } finally {
            lock.unlock();//释放锁
        }
    }
```

相比CopyOnWriteArrayList，ArrayList的add方法实现就显得啰嗦的多，而且ArrayList并不支持线程安全，至于为什么不支持线程安全，看代码就知道了，这几个调用的方法中都没有类似锁（与锁等效语义的组件）出现。下面再来看另一个版本的add方法：

```java
public void add(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        if (index > len || index < 0)
            throw new IndexOutOfBoundsException("Index: "+index+
                                                ", Size: "+len);
        Object[] newElements;
        int numMoved = len - index;
        if (numMoved == 0)
            newElements = Arrays.copyOf(elements, len + 1);
        else {
            newElements = new Object[len + 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index, newElements, index + 1,
                             numMoved);
        }
        newElements[index] = element;
        setArray(newElements);
    } finally {
        lock.unlock();
    }
}
```

在操作之前都是先 lock 住的，对于有效的 index，在复制时将原始数组分为两段，0 - (index - 1) 和 (index + 1) - size()，然后在 newElements 中对 index 位置的元素进行修改。

不论是哪个 add 方法，在修改完成数组后，都会调用 setArray，将原本的 array 指向新的数组。

```java
final void setArray(Object[] a) {
    array = a;
}
```

同样，在 set 方法中，都需要进行加锁，对原始数组进行复制，再对指定索引进行赋值，之后用 setArray 将原始数组指向新的数组。

```java
public E set(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        E oldValue = get(elements, index);

        if (oldValue != element) {
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len);
            newElements[index] = element;
            setArray(newElements);
        } else {
            // Not quite a no-op; ensures volatile write semantics
            setArray(elements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}
```

# 3、get 方法

```java
private E get(Object[] a, int index) {
    return (E) a[index];
}

public E get(int index) {
    return get(getArray(), index);
}
```

读操作允许多个线程共同访问，因此非常简单。

# 4、迭代器

内部迭代器类 COWIterator 如下，通过快照机制将 snapshot 指向当前的 array，并且在迭代过程中不支持 remove、set 和 add 操作，否则直接抛出异常。

CopyOnWriteArrayList 的迭代器 iterator 和 ArrayList 中的迭代器不同，ArrayList 中的迭代器支持迭代中的 remove 操作，CopyOnWriteArrayList 中有关数组修改的 add、set、remove 等操作均会直接抛出异常。

```java
static final class COWIterator<E> implements ListIterator<E> {
        /** Snapshot of the array */
        // 快照
        private final Object[] snapshot;
        /** Index of element to be returned by subsequent call to next.  */
        // 游标
        private int cursor;
        // 构造函数
        private COWIterator(Object[] elements, int initialCursor) {
            cursor = initialCursor;
            snapshot = elements;
        }
        // 是否还有下一项
        public boolean hasNext() {
            return cursor < snapshot.length;
        }
        // 是否有上一项
        public boolean hasPrevious() {
            return cursor > 0;
        }
        // next项
        @SuppressWarnings("unchecked")
        public E next() {
            if (! hasNext()) // 不存在下一项，抛出异常
                throw new NoSuchElementException();
            // 返回下一项
            return (E) snapshot[cursor++];
        }

        @SuppressWarnings("unchecked")
        public E previous() {
            if (! hasPrevious())
                throw new NoSuchElementException();
            return (E) snapshot[--cursor];
        }
        
        // 下一项索引
        public int nextIndex() {
            return cursor;
        }
        
        // 上一项索引
        public int previousIndex() {
            return cursor-1;
        }

        /**
         * Not supported. Always throws UnsupportedOperationException.
         * @throws UnsupportedOperationException always; {@code remove}
         *         is not supported by this iterator.
         */
        // 不支持remove操作
        public void remove() {
            throw new UnsupportedOperationException();
        }

        /**
         * Not supported. Always throws UnsupportedOperationException.
         * @throws UnsupportedOperationException always; {@code set}
         *         is not supported by this iterator.
         */
        // 不支持set操作
        public void set(E e) {
            throw new UnsupportedOperationException();
        }

        /**
         * Not supported. Always throws UnsupportedOperationException.
         * @throws UnsupportedOperationException always; {@code add}
         *         is not supported by this iterator.
         */
        // 不支持add操作
        public void add(E e) {
            throw new UnsupportedOperationException();
        }

        @Override
        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            Object[] elements = snapshot;
            final int size = elements.length;
            for (int i = cursor; i < size; i++) {
                @SuppressWarnings("unchecked") E e = (E) elements[i];
                action.accept(e);
            }
            cursor = size;
        }
}
```

# 5、优缺点

**优点：**

读读、读写可以同时进行，提升效率

从 CopyOnWriteArrayList 的名字就能看出它是满足 CopyOnWrite 的 ArrayList，CopyOnWrite 的意思是说，当容器需要被修改的时候，不直接修改当前容器，而是先将当前容器进行 Copy，复制出一个新的容器，然后修改新的容器，完成修改之后，再将原容器的引用指向新的容器。这样就完成了整个修改过程。

这样做的好处是，CopyOnWriteArrayList 利用了“不变性”原理，因为容器每次修改都是创建新副本，所以对于旧容器来说，其实是不可变的，也是线程安全的，无需进一步的同步操作。我们可以对 CopyOnWrite 容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素，也不会有修改。

CopyOnWriteArrayList 的所有修改操作（add，set等）都是通过创建底层数组的新副本来实现的，所以 CopyOnWrite 容器也是一种读写分离的思想体现，读和写使用不同的容器

**缺点：**

1.内存占用问题

因为 CopyOnWrite 的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，这一点会占用额外的内存空间。

在元素较多或者复杂的情况下，复制的开销很大

复制过程不仅会占用双倍内存，还需要消耗 CPU 等资源，会降低整体性能。

2.数据一致性问题

由于 CopyOnWrite 容器的修改是先修改副本，所以这次修改对于其他线程来说，并不是实时能看到的，只有在修改完之后才能体现出来。如果你希望写入的的数据马上能被其他线程看到，CopyOnWrite 容器是不适用的。

