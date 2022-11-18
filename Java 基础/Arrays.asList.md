首先，Arrays.asList 源码

```java
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```

参数为 T，泛型，因此当传入的是数组时，里面存储的类型是数组类型，得到的 size 为 1

```java
int[] a = {1, 2};
List<int[]> list = Arrays.asList(a);
System.out.println(list.size()); //输出 1
```

当使用 Arrays.asList 返回 List 集合时，为 ArrayList，不过注意的是，这里的 ArrayList 和平时使用的 ArrayList 不同，这里的 ArrayList 为 Arrays 的一个**私有静态内部类**，如下，该类继承自 AbstractList，但是没有对 add、remove 方法进行重写，在 abstract 中，add，remove 方法会直接抛出异常，但是重写了 set 方法，因此可使用 set 方法改变某些索引的值。这就是 Arrays.asList 得到的集合是不可变的原因。

```java
private static class ArrayList<E> extends AbstractList<E>
    implements RandomAccess, java.io.Serializable
{
    private static final long serialVersionUID = -2764017481108945198L;
    private final E[] a;

    ArrayList(E[] array) {
        a = Objects.requireNonNull(array);
    }

    @Override
    public int size() {
        return a.length;
    }

    @Override
    public Object[] toArray() {
        return a.clone();
    }

    @Override
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        int size = size();
        if (a.length < size)
            return Arrays.copyOf(this.a, size,
                                 (Class<? extends T[]>) a.getClass());
        System.arraycopy(this.a, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    @Override
    public E get(int index) {
        return a[index];
    }

    @Override
    public E set(int index, E element) {
        E oldValue = a[index];
        a[index] = element;
        return oldValue;
    }

    @Override
    public int indexOf(Object o) {
        E[] a = this.a;
        if (o == null) {
            for (int i = 0; i < a.length; i++)
                if (a[i] == null)
                    return i;
        } else {
            for (int i = 0; i < a.length; i++)
                if (o.equals(a[i]))
                    return i;
        }
        return -1;
    }

    @Override
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }

    @Override
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(a, Spliterator.ORDERED);
    }

    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        for (E e : a) {
            action.accept(e);
        }
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        E[] a = this.a;
        for (int i = 0; i < a.length; i++) {
            a[i] = operator.apply(a[i]);
        }
    }

    @Override
    public void sort(Comparator<? super E> c) {
        Arrays.sort(a, c);
    }
}
```

AbstractList 部分方法

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    ...
    
    public boolean add(E e) {
    	add(size(), e);
   		return true;
    }
    
    public void add(int index, E element) {
 	   throw new UnsupportedOperationException();
	}
    
    public E remove(int index) {
 	   throw new UnsupportedOperationException();
	}
    
    ...    
}
```