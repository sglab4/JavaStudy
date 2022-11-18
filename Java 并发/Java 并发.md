# 1、使用线程

使用线程的三种方法：

- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还**需要通过 Thread 来调用**。

## 实现 Runnable 接口

步骤：

1. 实现  Runnable接口中的 run 方法
2. 创建一个 Thread 实例，然后调用 Thread 实例的 start() 方法来启动线程。

在使用中也有不同的方法

```java
public class Test {
    public static void main(String[] args) {
        //1、原始方法
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable);
        thread.start();

        //2、匿名内部类
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("MyRunnable1");
            }
        });
        thread1.start();

        //3、Lambda 表达式
        Thread thread2 = new Thread(()->{
            System.out.println("MyRunnable2");
        });
        thread2.start();

        new Thread(()->{
            System.out.println("MyRunnable3");
        }).start();
    }
}

class MyRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println("MyRunnable");
    }
}
```

## 实现 Callable 接口

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

步骤：

1. 实现 Callable<> 中的 call 方法，call 方法需要有返回值，和 <> 中的返回值类型相同
2. 实现了 Callable 的类实例化后的对象作为 FutureTask 类构造方法的参数
3. FutureTask 实例的对象作为 Thread 构造方法的参数
4. Thread 开启线程
5. FutureTask 使用 get 方法获得 call 方法中的返回值，get 方法需要抛出异常

在具体使用中，有不同的方法

```java
public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //1
        MyCallable myCallable = new MyCallable();
        FutureTask<Integer> futureTask = new FutureTask<>(myCallable);
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println(futureTask.get()); //futureTask.get() 方法需要抛出异常

        //2
        FutureTask<Integer> futureTask1 = new FutureTask<>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                System.out.println("MyCallable1");
                return 100;
            }
        });
        new Thread(futureTask1).start();
        System.out.println(futureTask1.get());

        //3
        FutureTask<Integer> futureTask2 = new FutureTask<>(()->{
            System.out.println("MyCallable3");
            return 100;
        });
        new Thread(futureTask2).start();
        System.out.println(futureTask2.get());

    }
}

//返回值为 Integer 类型
class MyCallable implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        System.out.println("MyCallable");
        return 100;
    }
}
```

## 继承 Thread 类

继承自 Thread 类需要重写 run 接口，Thread 类也实现了 Runable 接口。

```java
public
class Thread implements Runnable {
....
}
```

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

同样在使用过程中也可有不同方法

```java
public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //1
        MyThread myThread = new MyThread();
        myThread.start();

        //2
        new MyThread().start();

        //3
        new Thread(new MyThread()).start();

    }
}

//返回值为 Integer 类型
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("MyThread");
    }
}
```

## Thread 和 Runnable 对比

**Tread类**：

- 继承Thread类来实现多线程
- 启动：子类对象.start()
- 不建议使用，避免单继承局限性

**Runnable类**：

- 接口Runnable实现多线程
- 启动：new Thread(目标对象).start()，采用代理
- 推荐使用，避免单继承局限性

此外，类可能只要求可执行就行，继承整个 Thread 类开销过大

# 2、基础线程机制

## Executor

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor：

- CachedThreadPool：一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

Executor 是一个接口，ExecutorService 接口继承自 Executor，Executors 是一个工具类，主要用于提供线程池相关的操作，用来创建 CachedThreadPool、FixedThreadPool、SingleThreadExecutor 的 ExecutorService 对象。

Executor 接口中只有 execute 类，接收的类型为 Runnable，用于将线程添加到线程池

```java
public interface Executor {
    void execute(Runnable command);
}
```

**为什么要用线程池:**

1. 减少线程创建和销毁的次数，使线程可以多次复用
2. 可以根据系统情况，调整线程的数量。防止创建过多的线程，消耗过多的内存（每个线程1M左右）

Java里面线程池的顶级接口是 Executor，但是严格意义上讲 Executor 并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是 ExecutorService。Executors 类，提供了一系列工厂方法用于创建线程池，返回的线程池都实现了 ExecutorService 接口。

ExecutorService 继承 Executor 接口，除了有 Executor 类外，还有如下类

![image-20210529210636833](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061642111.png)

```java
public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println("run");
                }
            });
        }
        //关闭线程池
        executorService.shutdown();
    }
}
```

## Daemon

线程分为**用户线程**和**守护线程**，虚拟机必须确保用户线程执行完毕，不用等待守护线程执行完毕，守护线程如操作日志，监控内存，垃圾回收等。

守护线程在后台为程序运行提供服务，当所有非守护线程结束后程序也会终止，同时会关闭所有守护线程。main() 属于非守护线程。

在线程启动之前使用 setDaemon() 方法可以将一个线程设置为守护线程。

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    Thread thread = new Thread(()->{
        System.out.println("test");
    });
    thread.setDaemon(true);//设置为守护线程
    thread.start();
}
```

## sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本线程进行处理。线程中抛出的其它异常也同样需要在本线程进行处理。

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    Thread thread = new Thread(()->{
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("test");
    });
    thread.start();
}
```

## yield()

调用静态方法 Thread.yield() 表示到目前为止可切换到其他线程，但该方法也只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

```java
public class TestYield {
    public static void main(String[] args) {
        new Thread(new MyYield(), "a").start();
        new Thread(new MyYield(), "b").start();
    }
}

class MyYield implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "线程开始");
        Thread.yield();
        System.out.println(Thread.currentThread().getName() + "线程结束");
    }
}
```

# 3、中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常会提前结束。

## InterruptedException

通过调用线程的 interrupt() 方法来中断该线程，如果该线程处于**阻塞、限期等待或者无限期等待**状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

```java
public class Test {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();
        //中断线程
        thread.interrupt();
        System.out.println("main");
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        try {
            sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("MyThread");
    }
}
```

![image-20210531201559273](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061642232.png)

##  interrupted()

若线程正在执行一个死循环，且没有 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法**不会使线程提前结束**。

但可使用 interrupt() 方法设置线程的中断标记，在一个线程的 run() 方法中，使用 interrupted() 方法，若调用线程的 interrupt() 方法，则 interrupted() 将会返回 true 值，从而达到控制线程的效果。

由于 interrupted() 方法和 interrupt() 方法均在 Thread 类中，因此使用线程的方法应该是继承 Thread 类

```java
public class Test {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();
        System.out.println("main");
        thread.interrupt();
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        while (!interrupted()) {

        }
        System.out.println("MyThread");
    }
}
```

## Executor 的中断操作

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 3; i++) {
        executorService.execute(()->{
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
    //调用每个线程的 interrupt() 方法
    executorService.shutdownNow();
}
```

![image-20210531203108353](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061642988.png)

# 4、互斥同步

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

## synchronized

### 1、同步一个代码块

```java
public void func() {
    synchronized (this) {
        // ...
    }
}
```

这里 this 意思是 synchronized 作用于**同一个对象**。多个线程调用同一个对象时，当一个线程进入同步语句块时，其他线程就必须等待。

```java
public class Test {
    public static void main(String[] args) {
        SynchronizedExample se = new SynchronizedExample();
        //由于调用同一个对象，因此当一个线程进入同步语句块时，另一个线程就必须等待。
        new Thread(()->{
            se.test();
        }).start();
        new Thread(()->{
            se.test();
        }).start();
    }

}

class SynchronizedExample {
    public void test() {
        synchronized (this) {
            for (int i = 1; i <= 5; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

```html
1 2 3 4 5 1 2 3 4 5 
```

当做用于不同对象时，则不会同步

```java
public class Test {
    public static void main(String[] args) {
        SynchronizedExample se1 = new SynchronizedExample();
        SynchronizedExample se2 = new SynchronizedExample();
        //不同对象，因此无需等待
        new Thread(()->{
            se1.test();
        }).start();
        new Thread(()->{
            se2.test();
        }).start();
    }

}

class SynchronizedExample {
    public void test() {
        synchronized (this) {
            for (int i = 1; i <= 20; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

### 2、同步一个方法

和同步代码块相同，均作用于**同一个对象**

```java
public synchronized void func () {
    // ...
}
```

### 3、同步一个类

```java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

作用于**整个类**，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

```java
public class Test {
    public static void main(String[] args) {
        SynchronizedExample se1 = new SynchronizedExample();
        SynchronizedExample se2 = new SynchronizedExample();
        //由于作用于整个类，因此也会同步
        new Thread(()->{
            se1.test();
        }).start();
        new Thread(()->{
            se2.test();
        }).start();
    }

}

class SynchronizedExample {
    public void test() {
        synchronized (SynchronizedExample.class) {
            for (int i = 1; i <= 20; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

### 4、同步一个静态方法

```java
public synchronized static void fun() {
    // ...
}
```

和同步一个类相同，作用于**整个类**，同一个类的不同对象调用该方法也会进行同步。

```java
public class Test {
    public static void main(String[] args) {
        SynchronizedExample se1 = new SynchronizedExample();
        SynchronizedExample se2 = new SynchronizedExample();
        //由于作用于整个类，因此也会同步
        new Thread(()->{
            se1.test();
        }).start();
        new Thread(()->{
            se2.test();
        }).start();
    }

}

class SynchronizedExample {
    public synchronized static void test() {
        for (int i = 1; i <= 20; i++) {
            System.out.print(i + " ");
        }
    }
}
```

## ReentrantLock

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

```java
public class Test {
    public static void main(String[] args) {
        LockDemo lockDemo = new LockDemo();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(()->{
            lockDemo.test();
        });
        executorService.execute(()->{
            lockDemo.test();
        });
    }
}

class LockDemo {
    private Lock lock = new ReentrantLock();
    public void test() {
        lock.lock();
        for (int i = 0; i < 10; i++) {
            System.out.print(i + " ");
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        lock.unlock();
    }
}
```

## 比较

### 1、锁的实现

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

### 2、性能

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

### 3、等待可中断

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

ReentrantLock 中的 lockInterruptibly() 方法使得线程可以在被阻塞时响应中断，比如一个线程t1通过 lockInterruptibly() 方法获取到一个可重入锁，并执行一个长时间的任务，另一个线程通过 interrupt() 方法就可以立刻打断t1线程的执行，来获取t1持有的那个可重入锁。而通过 ReentrantLock 的 lock() 方法或者 Synchronized 持有锁的线程是不会响应其他线程的 interrupt() 方法的，直到该方法主动释放锁之后才会响应 interrupt() 方法。

```java
public class ReentrantLockTest {
    static Lock lock1 = new ReentrantLock();
    static Lock lock2 = new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(new ThreadDemo(lock1, lock2));//该线程先获取锁1,再获取锁2
        Thread thread1 = new Thread(new ThreadDemo(lock2, lock1));//该线程先获取锁2,再获取锁1
        thread.start();
        thread1.start();
        thread.interrupt();//是第一个线程中断
    }

    static class ThreadDemo implements Runnable {
        Lock firstLock;
        Lock secondLock;
        public ThreadDemo(Lock firstLock, Lock secondLock) {
            this.firstLock = firstLock;
            this.secondLock = secondLock;
        }
        @Override
        public void run() {
            try {
                firstLock.lockInterruptibly();
                TimeUnit.MILLISECONDS.sleep(10);//更好的触发死锁
                secondLock.lockInterruptibly();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                firstLock.unlock();
                secondLock.unlock();
                System.out.println(Thread.currentThread().getName()+"正常结束!");
            }
        }
    }
}
```

### 4、公平锁

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

### 5、锁绑定多个条件

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

## 使用选择

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

synchronized：

- 优点：实现简单，语义清晰，便于 JVM 堆栈跟踪；加锁解锁过程由 JVM 自动控制，提供了多种优化方案。
- 缺点：不能进行高级功能（定时，轮询和可中断等）。

Lock：

- 优点：可定时的、可轮询的与可中断的锁获取操作，提供了读写锁、公平锁和非公平锁　
- 缺点：需手动释放锁 unlock，不适合 JVM 进行堆栈跟踪。

在一些内置锁无法满足需求的情况下，ReentrantLock 可以作为一种高级工具。当需要一些高级功能时才应该使用 ReentrantLock，这些功能包括：可定时的，可轮询的与可中断的锁获取操作，公平队列，以及非块结构的锁。否则，还是应该优先使用 Synchronized

# 5、线程之间的协作

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

##  join()

在一个线程中调用另一个线程的 join() 方法，会将当前线程挂起，调用另一个线程，直到另一个线程结束后才会继续执行当前线程。

使用 join() 方法强制执行另一个线程时，需要保证另一个线程已经处于 start 开启状态。

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(()->{
            System.out.println(Thread.currentThread().getName());
        }, "t1");
        Thread t2 = new Thread(()->{
            //在 t2 中调用 t1 的方法
            try {
                t1.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName());
        }, "t2");

        //先调用 t2 后调用 t1
        t2.start();
        //t1.start();
    }
}
```

对于以上程序来说，执行结果为 t2

```java
package test.basic;

public class Test {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(()->{
            System.out.println(Thread.currentThread().getName());
        }, "t1");
        Thread t2 = new Thread(()->{
            //在 t2 中调用 t1 的方法
            try {
                t1.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName());
        }, "t2");

        //先调用 t2 后调用 t1
        t2.start();
        Thread.sleep(1);
        t1.start();
    }
}
```

对于以上来说，由于 在执行 t2 过程中 t1 并未开启，因此执行结果为 t2  t1

##  wait(), notify(), notifyAll()

它们都属于 Object 的一部分，而不属于 Thread，只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

调用 wait() 方法时，线程在等待中被挂起，在挂起期间，该线程会释放锁，这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

```java
package test.basic;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        WaitNotify wn = new WaitNotify();
        //第一个线程
        executorService.execute(()->{
            wn.after();
        });
        //第二个线程，可见，wait() 方法会释放锁
        executorService.execute(()->{
            wn.after();
        });
        //第三个线程
        executorService.execute(()->{
            wn.before();
        });
        executorService.shutdown();
    }
}

class WaitNotify {
    public synchronized void before() {
        System.out.println("before");
        notifyAll();
    }

    public synchronized void after() {
        try {
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " after");
    }
}

```

![image-20210608212741993](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061642759.png)

notify() 和 notifyAll() 的区别在于 notify() 只会随机唤醒一个线程，而 notifyAll() 会唤醒所有 wait() 状态的线程。若将上述程序的 notifyAll() 改为 notify() 则结果为

![image-20210608212946891](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061642378.png)

由于线程 2 未被唤醒，处于 wait() 状态，因此程序并未结束。

# 6、线程状态

一个线程只能处于一种状态，这里所说的状态指的是 Java 虚拟中线程的状态。

## 新建（NEW）

创建后尚未启动。

## 可运行（RUNABLE）

正在 Java 虚拟机中运行。在操作系统层面，它可能处于运行状态，也可能等待资源调度，因此该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。

## 阻塞（BLOCKED）

请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其它线程已经占用了该 monitor lock，所以出于阻塞状态。要结束该状态进入从而 RUNABLE 需要其他线程释放 monitor lock。也就是需要等待其他线程释放锁

## 无限期等待（WAITING）

等待其它线程显式地唤醒。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取 monitor lock。而等待是主动的，通过调用 Object.wait() 等方法进入。

| 进入方法                                   | 退出方法                             |
| ------------------------------------------ | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | LockSupport.unpark(Thread)           |

## 限期等待（TIMED_WAITING）

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

| 进入方法                                 | 退出方法                                        |
| ---------------------------------------- | ----------------------------------------------- |
| Thread.sleep() 方法                      | 时间结束                                        |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法             | LockSupport.unpark(Thread)                      |
| LockSupport.parkUntil() 方法             | LockSupport.unpark(Thread)                      |

Thread.sleep()：称为“线程睡眠”

Object.wait()：使线程进入期限等待或无期限等待，称为“线程挂起”

睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

## 死亡（TERMINATED）

可以是线程结束任务之后自己结束，或者产生了异常而结束。

# 7、J.U.C - AQS

java.util.concurrent（J.U.C）大大提高了并发性能，AQS 被认为是 J.U.C 的核心。

## CountDownLatch

减法计数器。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，调用 await() 方法，await() 具有阻塞作用，直到计数器减为 0 才可继续向下执行。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643384.png)

```java
public static void main(String[] args) throws InterruptedException {
    final int totalThread = 5;

    CountDownLatch count = new CountDownLatch(totalThread);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < totalThread; i++) {
        executorService.execute(()->{
            System.out.println(Thread.currentThread().getName());
            //count.countDown();
        });
    }
    count.await();
    System.out.println("end");
    executorService.shutdown();
}
```

结果如下：

![image-20211205103604469](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643347.png)

由于 count 没有 countDown()，因此阻塞，不会向下执行。

将以上程序改为

```java
public static void main(String[] args) throws InterruptedException {
    final int totalThread = 5;

    CountDownLatch count = new CountDownLatch(totalThread);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < totalThread; i++) {
        executorService.execute(()->{
            System.out.println(Thread.currentThread().getName());
            count.countDown();
        });
    }
    count.await();
    System.out.println("end");
    executorService.shutdown();
}
```

结果如下：

![image-20210611204136340](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643380.png)

## CyclicBarrier

![](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643444.jpeg)

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，每次线程调用 await() 方法后计数器会减 1，并进行等待，当计数器为 0 时，所有调用 await() 等待的线程才可继续执行，并执行 barrierAction 一次。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。当 CyclicBarrier 减为 0 时，再次调用 await() 方法时，会更新为初始值 parties 重新减 1.

```java
public static void main(String[] args) throws InterruptedException {
    final int totalThread = 5;

    CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < totalThread; i++) {
        executorService.execute(()->{
            System.out.println("before..");
            try {
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("after..");
        });
    }
    executorService.shutdown();
}
```

![image-20210611210110263](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643002.png)

```java
public static void main(String[] args) throws InterruptedException {
    final int totalThread = 5;

    CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 3; i++) {
        executorService.execute(()->{
            System.out.println("before..");
            try {
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("after..");
        });
    }
    executorService.shutdown();
}
```

![image-20210611210223939](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643688.png)

```java
public static void main(String[] args) throws InterruptedException {
    final int totalThread = 2;

    CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 2 * totalThread; i++) {
        executorService.execute(()->{
            System.out.println("before..");
            try {
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("after..");
        });
    }
    executorService.shutdown();
}
```

![image-20210611210259796](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643931.png)

### CyclicBarrier 与 CountDownLatch 的区别

- CountDownLatch 的计数器只能使用一次，而 CyclicBarrier 的计数器可以使用 reset()方法进行重置，并且可以循环使用
- CountDownLatch 主要实现 1 个或 n 个线程需要等待其他线程完成某项操作之后，才能继续往下执行，描述的是 1 个或 n 个线程等待其他线程的关系。而 CyclicBarrier 主要实现了多个线程之间相互等待，直到所有的线程都满足了条件之后，才能继续执行后续的操作，描述的是各个线程内部相互等待的关系。
- CyclicBarrier 中提供了很多有用的方法，比如：可以通过 getNumberWaiting()方法获取阻塞的线程数量，通过 isBroken()方法判断阻塞的线程是否被中断。

## Semaphore

构造方法：

```java
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}
```

方法：

```java
Semaphore.acquire(); //获得一个许可，若已满，则线程等待
Semaphore.release(); //释放一个许可，然后唤醒等待的线程
Semaphore.availablePermits(); //返回 Semaphore 对象中当前可用的 permits 个数
Semaphore.drainPermits(); //返回可用的 permits 个数并将可用的 permits 个数置 0
```

作用：并发限流，控制最大的线程数

```java
public static void main(String[] args) throws InterruptedException {
    final int totalThread = 4;

    Semaphore semaphore = new Semaphore(totalThread);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 10; i++) {
        executorService.execute(()->{
            try {
                semaphore.acquire();
                System.out.print(semaphore.availablePermits() + " ");
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release();
            }
        });
    }
    executorService.shutdown();
}
```

![image-20210611211545395](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643311.png)

## Exchanger

用于两个线程交换数据，提供一个同步点 exchange，当线程到达同步点时，则会阻塞，一直等待另一个线程到达同步点进行交换数据。

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        Exchanger<String> exchanger = new Exchanger<>();
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                String s1 = "abcd";
                try {
                    exchanger.exchange(s1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                String s2 = "abcd";
                try {
                    String exchange = exchanger.exchange(s2);
                    System.out.println(exchange.equals(s2));
                    System.out.println(exchange);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

# 8、J.U.C - 其它组件

## FutureTask

FutureTask 可以用来获取 Callable 的返回值，FutureTask 继承自 RunnableFuture 接口，而 RunnableFuture 接口又继承自 Runnable 和 Future<V> 接口，因此，FutureTask 既可以当做一个任务执行，也可以有返回值。

```java
public class FutureTask<V> implements RunnableFuture<V>
```

```java
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask 可用于异步获取执行结果的场景，当一个任务执行时需要耗费较长时间时，可用 FutureTask 进行封装，主线程在完成自己的任务后在获取这个结果。

```java
package test.basic;

import java.util.concurrent.*;

public class Test {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        //futureTask 运行时间较长
        FutureTask<Integer> futureTask = new FutureTask<>(()->{
            int sum = 0;
            for (int i = 0; i < 100; i++) {
                sum = sum + i;
                Thread.sleep(10);
            }
            return sum;
        });
        new Thread(futureTask, "t1").start();

        //主线程执行自己的任务，相比 futureTask 运行时间更短
        new Thread(()->{
            System.out.println("t2 is running...");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "t2").start();
        System.out.println(futureTask.get());
    }
}
```

## BlockingQueue

java.util.concurrent.BlockingQueue 接口有以下阻塞队列的实现：

- **FIFO 队列** ：LinkedBlockingQueue、ArrayBlockingQueue（固定长度）
- **优先级队列** ：PriorityBlockingQueue（小顶堆）

以上队列提供了阻塞的 take() 和 put() 方法，若队列为空，则 take() 方法将阻塞取出，直到队列中有内容；若队列已满，则 put() 方法将阻塞放入，直到有空闲位置。

### 生产者消费者问题

使用 ArrayBlockingQueue 解决生产者消费者问题，在生产者中，若可使用 put() 方法，则表示可生产，输出 "produce"；在消费者中，若可使用 take() 方法，则表示可消费，输出 "consume"

```java
public class Test {
    private static final int MAX_NUM = 5;
    private static final int MAX_LOOP = 10;
    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(MAX_NUM);
    public static void main(String[] args) {
        ThreadPoolExecutor executorService = new ThreadPoolExecutor(5, MAX_LOOP, 100, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(MAX_NUM));
        for (int i = 0; i < MAX_LOOP; i++) {
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        queue.put("1");
                        // 有可能 put 之后线程挂起，因此可能会出现 1 2 2 这种情况
                        System.out.println("producer...");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        queue.take();
                        System.out.println("consumer...");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        executorService.shutdown();
    }
}
```

## ForkJoin

ForkJoin 主要用于并行计算中，将大的任务分成多个小任务进行并行计算。

![](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643083.jpeg)

fork 方法用于将新创建的子任务放入当前线程的 work queue 队列中，Fork/Join 框架将根据当前正在并发执行 ForkJoinTask 任务的 ForkJoinWorkerThread 线程状态，决定是让这个任务在队列中等待，还是创建一个新的 ForkJoinWorkerThread 线程运行它，又或者是唤起其它正在等待任务的 ForkJoinWorkerThread 线程运行它。

join 方法用于让当前线程阻塞，直到对应的子任务完成运行并返回执行结果。或者，如果这个子任务存在于当前线程的任务等待队列（work queue）中，则取出这个子任务进行“递归”执行。其目的是尽快得到当前子任务的运行结果，然后继续执行。 

使用 ForkJoin 的类需要继承抽象类 RecursiveTask<V>，实现 compute 方法。因为在之后使用 ForkJoinPool 启动 ForkJoin 时，ForkJoinPool 的 submit() 中的参数为 ForkJoinTask<T>，而 RecursiveTask<V> 继承自 ForkJoinTask<T> 类，因此可将自定义类作为 submit() 中的参数。

```java
public <T> ForkJoinTask<T> submit(ForkJoinTask<T> task){...}
```

```java
public abstract class RecursiveTask<V> extends ForkJoinTask<V>{
    protected abstract V compute();
}
```

在获得 ForkJoin 的计算结果时，需要使用 ForkJoinTask 来获得。

```java
package test.basic;

import java.util.concurrent.*;

public class Test extends RecursiveTask<Integer>{
    private final int threshold = 5;
    int first;
    int last;

    public Test(int first, int last) {
        this.first = first;
        this.last = last;
    }

    @Override
    protected Integer compute() {
        int sum = 0;
        //当较小时，普通 for 循环
        if (last - first <= threshold) {
            for (int i = first; i <= last; i++) {
                sum = sum + i;
            }
        } else { //使用 ForkJoin
            int mid = first + (last - first) / 2;
            Test leftForkJoin = new Test(first, mid);
            Test rightForkJoin = new Test(mid + 1, last);
            leftForkJoin.fork();
            rightForkJoin.fork();
            sum = leftForkJoin.join() + rightForkJoin.join();
        }
        return sum;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //使用 ForkJoinPool 开启
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        //使用 ForkJoinTask 获得 ForkJoin 的结果
        ForkJoinTask<Integer> forkJoinTask = forkJoinPool.submit(new Test(1, 100));
        System.out.println(forkJoinTask.get());
    }
    
}
```

ForkJoin 使用工作窃取提高了 CPU 的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争。

如下图，Thread2 从 Thread1 的队列中拿出最晚的 Task1 任务，Thread1 会拿出 Task2 来执行，这样就避免发生竞争。但当队列中只有一个任务时还是会发生竞争。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643473.png)

# 9、线程不安全示例

若多个线程对同一个共享变量进行访问而不采取同步操作的话，可能会产生并发问题，得到操作不一致的结果。

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {
    public static void main(String[] args) throws InterruptedException {
        final int threadSize = 1000;
        ExecutorService executorService = Executors.newCachedThreadPool();
        //引入计数器
        final CountDownLatch count = new CountDownLatch(threadSize);
        TestUnsafeDemo t = new TestUnsafeDemo();
        for (int i = 0; i < threadSize; i++) {
            executorService.execute(()->{
                t.add();
                count.countDown();
            });
        }
        count.await();
        executorService.shutdown();
        System.out.println(t.get());
    }
}

class TestUnsafeDemo {
    private int cnt = 0;

    public void add() {
        cnt++;
    }

    public int get() {
        return cnt;
    }
}
```

![image-20210617213756667](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643758.png)

# 10、Java 内存模型

Java 内存模型试图屏蔽各种硬件和操作系统的内存访问差异，使得 Java 程序在各种平台下可达到一致的内存访问效果。

## 主内存与工作内存

由于处理器上寄存器的读写速度远高于内存，因此在处理器和内存间加入了高速缓存。但加入高速缓存后会引入另一个问题：缓存一致性，即多个高速缓存共享一块主内存，这些缓存中的数据可能会不一致，因此需要一些解决这些问题。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643168.png)

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643736.png)

## 内存间交互操作

Java 内存模型定义了 8 种操作来完成主内存和工作内存之间的交互操作。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643973.png)

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：作用于主内存的变量，加锁
- unlock：作用于主内存的变量，释放锁

## 内存模型的三大特性

### 1、原子性

原子性分为处理器实现原子性和 JMM 实现原子性，处理器实现原子性的操作有总线锁定和缓存锁定，JMM 实现原子性的操作有加锁 synchronized 和 CAS。

Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock **单操作**具有原子性，但是在 32 位处理器上，对于没有被 volatile 修饰的 64 位数据（long，double）的读写操作将会被划分为 32 位操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

虽然单个操作是具有原子性，但是在多线程中可能会出现问题，例如在（9 线程不安全示例）中 cnt 不等于 1000。

为了方便讨论，将内存操作简化为 load、assign、store。

如下图，两个线程同时对 cnt 进行操作，由于 T1 将 cnt 值加一后还未 store 到 主内存中，因此 T2 可能获得未被更新过的值。从逻辑来看，两个线程虽然对 cnt 进行了两次自加操作，但是由于 T2 读取的值为 T1 未更新的值，因此 cnt 最终的值为 1 而不是 2。可发现，int 操作只是 load、assign、store 操作具有原子性，而多线程执行的时候变可能会出现问题。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643455.jpeg)

若要解决以上问题，需要使用原子类 AtomicInteger 类，来保证多个线程修改变量的原子性。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643893.jpeg)

使用 AtomicInteger 对（9 线程不安全示例）重写，如下

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

public class Test {
    public static void main(String[] args) throws InterruptedException {
        final int threadSize = 1000;
        ExecutorService executorService = Executors.newCachedThreadPool();
        //引入计数器
        final CountDownLatch count = new CountDownLatch(threadSize);
        TestUnsafeDemo t = new TestUnsafeDemo();
        for (int i = 0; i < threadSize; i++) {
            executorService.execute(()->{
                t.add();
                count.countDown();
            });
        }
        count.await();
        executorService.shutdown();
        System.out.println(t.get());
    }
}

class TestUnsafeDemo {
    private AtomicInteger cnt = new AtomicInteger();

    public void add() {
        cnt.getAndIncrement();
    }

    public int get() {
        return cnt.get();
    }
}
```

除了使用原子类外，还可使用 synchronized 锁和 lock 锁实现操作的原子性，在虚拟机实现上对应的字节码指令为 monitorenter 和 monitorexit。

```java
class TestUnsafeDemo {
    private int cnt = 0;

    public synchronized void add() {
    	cnt++;
    }

    public int get() {
        return cnt;
    }
}
```

```java
class TestUnsafeDemo {
    private int cnt = 0;

    Lock lock = new ReentrantLock();

    public void add() {
        lock.lock();
        cnt++;
        lock.unlock();
    }

    public int get() {
        return cnt;
    }
}
```

### 2、可见性

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型实现可见性的方法是当共享变量被修改同步到主内存后，在变量读取前从主内存刷新变量值来实现可见性。

三种实现**共享变量**可见性的方式：

- volatile
- synchronized，Lock：对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

对上例中的 cnt 加 volatile 并不能解决操作可见性的问题。

### 3、有序性

有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了**指令重排序**。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，对 volatile 变量读及其之后不可重排，对 volatile 变量的写及其之间不可重排，也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

## 先行发生原则

先行发生原则，让一个操作无需控制就能先于另一个操作完成。

### 1. 单一线程原则

在一个线程内，在程序前面的操作先行发生于后面的操作。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643351.png)

### 2. 管程锁定规则

一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643851.png)

### 3. volatile 变量规则

对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643848.png)

###  4. 线程启动规则

Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061643704.png)

###  5. 线程加入规则

join() 方法返回先行发生于 Thread 对象的结束。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061644233.webp)

### 6. 线程中断规则

对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，也就是说，在主动调用 interrupt() 中断线程之前，线程未出现代码检测中断。可以通过 interrupted() 方法检测到是否有中断发生。

### 7. 对象终结规则

一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

### 8. 传递性

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

# 11、线程安全

线程安全有以下几种实现方式：

## 不可变

不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：

- final 修饰的基本数据类型
- String
- 枚举类型
- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。

对于集合类型，可以使用 Collections.unmodifiableXXX() 方法来获取一个不可变的集合。如下程序中，unmodifiableMap 为不可变集合。

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        Map<String, Integer> map = new HashMap<>();
        //unmodifiableMap 为不可变集合
        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(map);
        unmodifiableMap.put("a", 1);
    }
}
```

```java
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.Collections$UnmodifiableMap.put(Collections.java:1457)
	at Test.main(Test.java:16)
```

unmodifiableMap 对象的 put 方法源码如下，Collections.unmodifiableXXX() 先对原始的集合进行拷贝，需要对集合进行修改的方法都直接抛出异常。

```java
public V put(K key, V value) {
    throw new UnsupportedOperationException();
}
```

##  互斥同步

synchronized 和 ReentrantLock。

## 非阻塞同步

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

基于**冲突检测**的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

### 1、CAS

乐观锁需要操作和冲突检测这两个步骤具有原子性，不再采用互斥同步来完成，只能靠硬件完成，硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

### 2、AtomicInteger

整数原子类 AtomicInteger 的 incrementAndGet() 方法调用了 Unsafe 类的 getAndAddInt()  CAS 操作。

```java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

Unsafe 类的 getAndAddInt() 源码如下，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。var5 的内存地址通过 getIntVolatile(var1, var2) 方法，为 var1 + var2。compareAndSwapInt(var1, var2, var5, var5 + var4) 方法的四个参数中，第一个参数是指定哪个对象，第二个参数是指 var1 对象中的偏移量（内存地址，用于定位某个内存），第三个参数是期望值，第四个参数是更新值。此方法的含义是判断var1对象内存的var2位置存储的值是否为var5，若是，那么就更新内存地址为 var1 + var2 的变量为 var5 + var4。

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。

### 3、ABA

如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。这便是 ABA 问题

J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步（synchronized 、 ReentrantLock）可能会比原子类更高效。

AtomicStampedReference 构造方法：

```java
public AtomicStampedReference(V initialRef, int initialStamp) //initialRef 为初始值，initialStamp 为初始版本号
```

compareAndSet(V   expectedReference,  V   newReference,  int expectedStamp, int newStamp) 方法如下，expectedReference 为预期值，newReference 为新值，expectedStamp 为预期版本号，newStamp 为新版本号。

```java
public boolean compareAndSet(V   expectedReference,
                             V   newReference,
                             int expectedStamp,
                             int newStamp) {
    Pair<V> current = pair;
    return
        expectedReference == current.reference &&
        expectedStamp == current.stamp &&
        ((newReference == current.reference &&
          newStamp == current.stamp) ||
         casPair(current, Pair.of(newReference, newStamp)));
}
```

## 无同步方案

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

### 1、栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于**线程私有**的。

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        StackExample se = new StackExample();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(()->{se.test();});
        executorService.execute(()->{se.test();});
        executorService.shutdown();
    }
}

class StackExample {
    public void test() {
        int cnt = 0;
        for (int i = 0; i < 100; i++) {
            cnt = cnt + i;
        }
        System.out.println(cnt);
    }
}
```

```html
4950
4950
```

### 2、线程本地存储（Thread Local Storage）

#### 简介

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果在同一个线程中执行，那么将不会发生并发问题，不需要同步也能保证线程之间不出现数据争用的问题。

符合以上特点的应用有

- 消费队列的架构模式（如“生产者-消费者”模式），将产品的消费过程在同一个线程中消费完
- 经典 Web 交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

#### ThreadLocal

参考 https://www.cnblogs.com/dolphin0520/p/3920407.html

ThreadLocal 为变量在每个线程中都创建了一个副本，每个线程可以访问自己内部的副本变量。接下来进行说明。

ThreadLocal 类提供了下面几个方法：

```java
public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }
```

get()方法是用来获取ThreadLocal在当前线程中保存的变量副本，set()用来设置当前线程中变量的副本，remove()用来移除当前线程中变量的副本，initialValue()是一个protected方法，一般是用来在使用时进行重写的，它是一个延迟加载方法。

> get() 方法

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

在以上源码中，先获得当前线程，然后通过 getMap(t) 方法获取到一个 map，map 的类型为 ThreadLocalMap。然后接着下面获取到 <key,value> 键值对，**注意这里获取键值对传进去的是 this，而不是当前线程 t**，this 指代的是当前的 ThreadLocal，也就是说，**ThreadLocalMap 的键值类型为 ThreadLocal**。若获取成功，则返回 value 值，否则通过 setInitialValue() 方法返回 value 值。

接下来看上述方法中的 getMap() 方法

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

在线程类 Thread 中，有成员变量 threadLocals，类型为 ThreadLocalMap, getMap 方法获得了线程 t 的 threadLocals

>setInitialValue() 方法

```java
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

很容易了解，就是如果 map 不为空，就设置键值对，否则创建 Map。

> set() 方法

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

若当前线程的 threadLocals 为空，则设置键值对，否则创建创建 Map。

> 总结

ThreadLocal 为每个线程创建副本的过程：

首先，在每个线程Thread内部有一个静态内部类 ThreadLocal.ThreadLocalMap 类型的成员变量 threadLocals，这个 threadLocals 就是用来存储实际的变量副本的，键值为当前 ThreadLocal 变量，value 为变量副本（即T类型的变量）。

初始时，在 Thread 里面，threadLocals 为空，当通过 ThreadLocal 变量调用 get() 方法或者 set() 方法，就会对Thread 类中的 threadLocals 进行初始化，并且以当前 ThreadLocal 变量为键值，以 ThreadLocal 要保存的副本变量为 value，存到 threadLocals。

然后在当前线程里面，如果要使用副本变量，就可以通过 get 方法在 threadLocals 里面查找。

ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争。在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险。

### 3. 可重入代码（Reentrant Code）

这种代码也叫做纯代码（Pure Code），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

# 12、锁优化

这里的锁优化主要是指 JVM 对 synchronized 的优化。锁优化部分详细可看 方腾飞《Java 并发编程艺术》

## 自旋锁

由于 synchronized 会使其他线程进入阻塞状态，因此应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。根据这个特性，自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

虽然自旋锁能够避免线程进入阻塞状态从而减少开销，但是它所进行的自旋操作却占用了 CPU 的时间，因此，自旋锁只适应于共享数据的锁定状态很短的场景。

在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

## 锁消除

锁消除是指对于被检测出**不可能存在竞争的共享数据**的锁进行消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。

对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除。锁消除的主要判定依据来源于逃逸分析的数据支持，如果判断在一段代码中，堆上的所有数据都不会逃逸出去从而被其他线程访问到，那就可以把它们当做栈上数据对待，认为它们是线程私有的，同步加锁自然就无须进行。

一些方法隐式的加了很多锁。例如如下字符串拼接方法：

```java
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```

String 是一个不可变的类，编译器会对 String 的拼接自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append() 操作：

```java
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

StringBuffer 的 append 方法有一个同步块 synchronized，虚拟机观察变量 sb，很快就会发现它的动态作用域被限制在 concatString() 方法内部。其他线程无法访问到它，因此可以进行消除。

## 锁粗化

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

锁粗化的思想是把加锁的范围扩展（粗化）到整个操作序列的外部，尽量减少加锁和解锁的操作步骤。对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

## 轻量级锁

JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无锁状态（unlocked）、偏向锁状态（biasble）、轻量级锁状态（lightweight locked）和重量级锁状态（inflated）。

 HotSpot 虚拟机对象头的内存布局，这些数据被称为 Mark Word。其中 tag bits 对应了五个状态，这些状态在右侧的 state 表格中给出。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061644156.png)

下图左侧是一个线程的虚拟机栈，其中有一部分称为 Lock Record 的区域，这是在轻量级锁运行过程创建的，用于存放锁对象的 Mark Word。而右侧就是一个锁对象，包含了 Mark Word 和其它信息。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061644098.png)

轻量级锁是相对于传统的重量级锁而言，它使用 **CAS 操作**来避免重量级锁使用互斥量的开销。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都使用互斥量进行同步，可以先采用 CAS 操作进行同步，如果 CAS 失败了再改用互斥量进行同步。

当尝试获取一个锁对象时，如果锁对象标记为 01，说明锁对象的锁未锁定（unlocked）状态。此时虚拟机在当前线程的虚拟机栈中创建 Lock Record，然后使用 CAS 操作将对象的 Mark Word 更新为 Lock Record 指针。如果 CAS 操作成功了，那么线程就获取了该对象上的锁，并且对象的 Mark Word 的锁标记变为 00，表示该对象处于轻量级锁状态。

如果 CAS 操作失败了，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的虚拟机栈，如果是的话说明当前线程已经拥有了这个锁对象，那就可以直接进入同步块继续执行，否则说明这个锁对象已经被其他线程线程抢占了。如果有两条以上的线程争用同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁。

## 偏向锁

偏向锁的思想是偏向于第一个获取锁对象的线程，第一个获取锁对象的线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。

当锁对象第一次被线程获得的时候，进入偏向状态，标记为 1 01。同时使用 CAS 操作将线程 ID 记录到 Mark Word 中，如果 CAS 操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。

当有另外一个线程去尝试获取这个锁对象时，偏向状态就宣告结束，此时撤销偏向（Revoke Bias）后恢复到未锁定状态或者轻量级锁状态。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208061644545.jpeg)

# 13、线程间的通信方式

线程通信主要可以分为三种方式，分别为**共享内存**、**消息传递**和**管道流**。

- 共享内存：volatile 共享内存
- 消息传递：wait/notify 或 join
- 管道流：管道流的输入输出

# 14、多线程开发良好的实践

- 命名时做到见名知意
- 缩小同步范围，从而减少锁争用。例如对于 synchronized，应该尽量使用同步块而不是同步方法。
- 多使用同步工具类，少使用 wait() 和 notify()。首先，CountDownLatch, CyclicBarrier, Semaphore 和 Exchanger 这些同步类简化了编码操作，而用 wait() 和 notify() 很难实现复杂控制流；其次，这些同步类是由最好的企业编写和维护，在后续的 JDK 中还会不断优化和完善。
- 使用 BlockingQueue 实现生产者消费者问题。
- 多用并发集合少用同步集合，例如应该使用 ConcurrentHashMap 而不是 Hashtable。
- 使用本地变量和不可变类来保证线程安全。
- 使用线程池而不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地利用有限的线程来启动任务。
