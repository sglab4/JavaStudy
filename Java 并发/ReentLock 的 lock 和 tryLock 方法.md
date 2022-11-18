首先明确继承关系，ReentrantLock 继承自 Lock，需重写 lock、tryLock 和 unLock 等方法，Sync 是 ReentLock 的内部类，继承自 AQS，公平锁和非公平锁继承自 Sync。

根据源码，当 ReentLock 调用 lock 方法时，将会调用内部类 Sync 的 lock 方法

```java
public void lock() {
    sync.lock();
}
```

而 Sync 的 lock 方法又由内部类公平锁 FairSync 和非公平锁 NonfairSync 实现

![image-20220102164925590](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202201021649618.png)

以非公平锁为例，调用非公平锁的 lock 方法后将会调用 CAS 将 AQS 的 state 变量写为 1，否则调用 AQS 的 acquire 方法，基于同步队列实现。

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

当 ReentLock 调用 tryLock 方法时，会调用非公平锁的 nonfairTryAcquire 方法，在 nonfairTryAcquire 方法中，先得到同步状态 state，然后查看 state 是否等于 0，若等于 0 则表示可获取同步状态；否则查看当前线程是否为获取同步状态的线程，若是则将 state + 1，这也是“可重入” 的体现。若以上两种情况均不满足，则直接返回 false。

```java
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

综上所述，lock 方法会调用 CAS 设置 state，也涉及到了判断当前线程是否为已获得同步状态的线程，若使用 CAS 获取同步状态失败，则会调用 AQS 的 acquire 方法使用同步队列维护同步状态。而 tryLock 方法则会尝试获取同步状态，若获取同步状态失败，则直接返回 false，不会使用 AQS 的同步队列将该线程阻塞。

