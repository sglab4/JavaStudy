根据《Java 并发编程艺术》和源码，读写锁中维护了 Sync 锁和一个读锁 ReadLock 和写锁 WriteLock

# 读锁 ReadLock

lock 方法源码如下

```java
public void lock() {
    sync.acquireShared(1);
}
```

调用的是 Sync 即 AQS 的共享加锁方法

tryLock 方法源码如下

```java
public boolean tryLock() {
    return sync.tryReadLock();
}

....
    
final boolean tryReadLock() {
    Thread current = Thread.currentThread();
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
            return false;
        int r = sharedCount(c);
        if (r == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return true;
        }
    }
}
```

分析见书中 P144，简单来说，读锁中只维护了所有线程获取读锁的次数总和，每个线程各自获得读锁的次数被维护在各自的 ThreadLocal 中。

# 写锁 WriteLock

lock 源码如下

```java
public void lock() {
    sync.acquire(1);
}
```

调用 AQS 排他锁的 acquire 结合双向队列实现同步

tryLock 源码如下

```java
public boolean tryLock( ) {
    return sync.tryWriteLock();
}

...

final boolean tryWriteLock() {
    Thread current = Thread.currentThread();
    int c = getState();
    if (c != 0) {
        int w = exclusiveCount(c);
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
    }
    if (!compareAndSetState(c, c + 1))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

详细说明见书中 P143



