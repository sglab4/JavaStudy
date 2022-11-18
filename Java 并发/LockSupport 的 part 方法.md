参考：

- https://www.cnblogs.com/yonghengzh/p/14280670.html
- https://cloud.tencent.com/developer/article/1645909

park/unpark 为 Lock（AQS）提供了挂起/恢复当前线程的能力，在调用 park 后，线程进入等待状态，在这一期间，线程不会释放资源。

每个线程都会关联一个 Parker 对象，每个 Parker 对象都各自维护了三个角色：计数器、互斥量、条件变量。

> **park 和 unpark 方法总结**

**park 操作：**

1. 获取当前线程关联的 Parker 对象。

2. 将计数器置为 0，同时检查计数器的原值是否为 1，如果是则放弃后续操作。

   首先利用一个原子交换操作将计数器的值改为 0，同时检查计数器的原值是否大于 0，如果大于 0，表示当前 Parker 对象的 unpark 方法先于 park 方法执行了（因为 unpark 方法会把计数器的值改为 1），那么本次 park 方法将直接返回，表示取消本次操作。如果计数器的原值不大于 0，则继续往下执行。

3. 在互斥量上加锁。

   接着判断当前线程是否被标记了中断，如果是的话就直接返回，否则就通过 `pthread_mutex_trylock` 函数尝试加 mutex 锁，如果加锁失败也直接返回。（`pthread_mutex_trylock` 函数是一个系统调用，它会针对操作系统的一个互斥量进行加锁，加锁成功将返回 0）。

4. 在条件变量上阻塞，同时释放锁并等待被其他线程唤醒，当被唤醒后，将重新获取锁。

5. 当线程恢复至运行状态后，将计数器的值再次置为 0。

6. 释放锁。

**unpark 操作：**

1. 获取目标线程关联的 Parker 对象（注意目标线程不是当前线程）。
2. 在互斥量上加锁。
3. 将计数器置为 1。
4. 唤醒在条件变量上等待着的线程。
5. 释放锁。

先是给当前线程加锁，然后将计数器的值改为 1，接着判断 Parker 对象所关联的线程是否被 park，如果是，则通过 `pthread_mutex_signal` 函数唤醒该线程，最后释放锁。

当线程恢复运行后，计数器的值会再次被置为 0，然后线程会释放锁，并结束整个 park 操作。

>**park/unpark 和 wait/notify区别**

```java
public class LockSyncTest {
    private static Object lock = new Object();
    //保存调用park的线程，以便后续唤醒
    private static Thread parkedThread;

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(()->{
             synchronized (lock){
                 System.out.println("unpark前");
                 LockSupport.unpark(parkedThread);
                 System.out.println("unpark后");
             }
        });

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                //和t1线程用同一把锁时，park不会释放锁资源，若换成this锁，则会释放锁
                synchronized (lock){
                    System.out.println("park前");
                    parkedThread = Thread.currentThread();
                    LockSupport.park();
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("park后");
                }
            }
        });

        t2.start();
        Thread.sleep(100);
        t1.start();

    }
}
```

以上程序中，会一直卡在 t2 线程，因为 park 方法后不会释放资源，t1 无法执行

若将 t2 中的 lock 换为 this，则 t1 将会正常执行，唤醒等待的线程。

1) wait 和 notify 方法必须和同步锁 synchronized 一块儿使用。而 park/unpark 使用就比较灵活了，没有这个限制，可以在任何地方使用。

2) park/unpark 使用时没有先后顺序。而 wait 必须在 notify 前先使用，如果先 notify，再 wait，则线程会一直等待。

3) notify 释放等待队列中队首的一个线程，并不能指定某个特定线程，notifyAll 是释放锁对象中的所有线程。而 unpark 方法可以唤醒指定的线程。

4) 调用 wait 方法会使当前线程释放锁资源，但使用的前提是必须已经获得了锁。而 park 不会释放锁资源。

