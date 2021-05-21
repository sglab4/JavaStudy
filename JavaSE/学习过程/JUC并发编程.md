[TOC]

# 概述

JUC其实是Java下的java.util.concurrent，java.util.concurrent.atomic，java.util.concurrent.locks三个包

Java默认有2个线程，main和GC垃圾回收线程

Java创建线程的三种方式：Thread、Runnable、Callable

问：Java能直接开启线程吗？

答：不能，根据源码：开启线程最终调用的是start0()方法，使用C++，Java无法直接操作硬件，在JVM虚拟机上运行

```java
private native void start0();
```

并发和并行概念：

并发：多个线程同时操作一个资源

并行：CPU多核，多个线程同时进行

并发编程本质：**充分利用CPU资源**

查看CPU的核数：

```java
System.out.println(Runtime.getRuntime().availableProcessors());
```

线程状态：

```java
public enum State {
	//创建
    NEW,
	//运行
    RUNNABLE,
	//阻塞
    BLOCKED,
	//等待，一直等
    WAITING,
	//等待，过期不候
    TIMED_WAITING,
	//终止
    TERMINATED;
}
```

wait和sleep区别：

1. wait来自Object类，sleep来自Thread类
2. wait会释放锁，sleep不会释放锁
3. **wait必须在同步代码块中使用**，sleep可在任何地方时使用
4. wait不需要捕获异常，sleep必须捕获异常

# Synchronized锁

线程就是一个单独的资源类，没有任何附属操作，拿来即用

Synchronized：其实是一种**等待机制**，多个需要访问此对象的线程进入到该对象的等待池，形成队列，等待前面线程执行完后下一个线程再使用。

开启线程时，使用：

```java
new Thread(()->{
    //run中的方法
    xxx
}).start();
```

# Lock锁

Lock 接口的实现允许锁在不同的作用范围内获取和释放，并允许以任何顺序获取和释放多个锁

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210311024308Lock.JPG)

接口Lock的所有实现类：

- ReentrantLock 可重入互斥锁
- ReentrantReadWriteLock.ReadLock 读锁
- ReentrantReadWriteLock.WriteLock 写锁

ReentrantLock构造方法：

```java
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

分为公平锁FairSync()和非公平锁NonfairSync()

- 公平锁：先来后到
- 非公平锁：可插队，默认为非公平锁

------

**Synchronized和Lock锁的区别**：

1. Synchronized是内置关键字，而Lock是Java接口
2. Synchronized无法判断锁的状态，Lock可判断是否获取了锁
3. Synchronized自动释放锁，而Lock需手动释放
4. Synchronized 线程1获得锁，线程2一直等待，Lock就不会一直等待下去（Lock.tryLock()）
5. Synchronized可重入锁、不可中断、非公平，Lock可重入锁，可判断是否中断，可公平或非公平锁
6. Synchronized锁少量代码，Lock锁大量代码块

# 生产者消费者问题

## Synchronized

```java
package com.juc;

public class SynPro {
    public static void main(String[] args) {
        Test test = new Test();

        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    test.increase();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "A").start();

            new Thread(()->{
                try {
                    test.decrease();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "B").start();

            new Thread(()->{
                try {
                    test.increase();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "C").start();

            new Thread(()->{
                try {
                    test.decrease();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "D").start();




        }


    }
}

class Test {
    private int num = 0;

    //+1
    public synchronized void increase() throws InterruptedException {
        if (num != 0) {
            this.wait();
        }
        num++;
        System.out.println(Thread.currentThread().getName() + "-->" + num);
        this.notifyAll();
    }

    //-1
    public synchronized void decrease() throws InterruptedException {
        if (num == 0) {
            this.wait();
        }
        num--;
        System.out.println(Thread.currentThread().getName() + "-->" + num);
        this.notifyAll();
    }
}
```

可能会存在**虚假唤醒**问题，将**wait()放到while循环**中

```java
public synchronized void increase() throws InterruptedException {
    while (num != 0) {
        this.wait();
    }
    num++;
    System.out.println(Thread.currentThread().getName() + "-->" + num);
    this.notifyAll();
}

//-1
public synchronized void decrease() throws InterruptedException {
    while (num == 0) {
        this.wait();
    }
    num--;
    System.out.println(Thread.currentThread().getName() + "-->" + num);
    this.notifyAll();
}
```

## Lock

synchronize -- wait -- notify/notifall

Lock -- await -- signal

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210311033500Condition2.JPG)

其中，await和signal均为接口Condition中的方法，可使用Lock.newCondition()创建Condition的实现类

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210311033358Condition.JPG)

对Synchronized中修改

```java
package com.juc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockPro {

    public static void main(String[] args) {
        Test2 test = new Test2();

        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    test.increase();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "A").start();

            new Thread(()->{
                try {
                    test.decrease();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "B").start();

            new Thread(()->{
                try {
                    test.increase();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "C").start();

            new Thread(()->{
                try {
                    test.decrease();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "D").start();

        }


    }
}

class Test2 {
    private int num = 0;

    //+1
    public void increase() throws InterruptedException {
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();
        try {
            lock.lock();
            //业务代码
            while (num != 0) {
                condition.await();
            }
            num++;
            System.out.println(Thread.currentThread().getName() + "-->" + num);
            condition.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    //-1
    public void decrease() throws InterruptedException {
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();
        try {
            lock.lock();
            //业务代码
            while (num == 0) {
                condition.await();
            }
            num--;
            System.out.println(Thread.currentThread().getName() + "-->" + num);
            condition.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }
}
```

## Condition精准通知唤醒

若要求以上问题有序执行，即A,B,C，可实现**精准唤醒**

```java
package com.juc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

//有序执行，A->B，B->C，C->A
public class LockPro2 {
    public static void main(String[] args) {
        Test3 test3 = new Test3();

        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                test3.printA();
            }
        }, "A").start();

        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                test3.printB();
            }
        }, "B").start();

        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                test3.printC();
            }
        }, "C").start();

    }
}

class Test3 {
    private Lock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();
    Condition condition3 = lock.newCondition();
    private int flag = 1; //1为A，2为B，3为C

    public void printA() {
        try {
            lock.lock();
            while (flag != 1) {
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName() + "->A");
            flag = 2;
            condition2.signal(); //唤醒condition2,B，精准唤醒
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        try {
            lock.lock();
            while (flag != 2) {
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName() + "->B");
            flag = 3;
            condition3.signal(); //唤醒condition3,C，精准唤醒
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() {
        try {
            lock.lock();
            while (flag != 3) {
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName() + "->C");
            flag = 1;
            condition1.signal(); //唤醒condition1,A，精准唤醒
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

# 锁

锁的是方法的调用者，先获得锁，先执行

```java
package com.juc;

import java.util.concurrent.TimeUnit;

public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            //先获得锁，先执行
            try {
                phone.call();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "A").start();

        TimeUnit.SECONDS.sleep(1); //sleep 1秒

        new Thread(()->{
            phone.sendMs();
        }, "B").start();

    }
}

//锁的是方法的调用者，在本例中，锁的是phone，谁先拿到谁先执行
class Phone {
    public synchronized void sendMs() {
        System.out.println("sendMs");
    }
    public synchronized void call() throws InterruptedException {
        TimeUnit.SECONDS.sleep(1);
        System.out.println("call");
    }
}
```

输出结果为：

call    

sendMs

将以上代码进行修改，加入一个普通方法

```java
package com.juc;

import java.util.concurrent.TimeUnit;

public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            phone.sendMs();
        }, "A").start();

        //TimeUnit.SECONDS.sleep(1); //sleep 1秒

        new Thread(()->{
            phone.hello();
        }, "B").start();

    }
}

//锁的是对象的调用者，在本例中，锁的是phone，谁先拿到谁先执行
class Phone {
    public synchronized void sendMs() {
        System.out.println("sendMs");
    }
    public synchronized void call() throws InterruptedException {
        TimeUnit.SECONDS.sleep(1);
        System.out.println("call");
    }
    public void hello () {
        System.out.println("hello");
    }
}
```

输出可能先为sendMs，也可能先为hello，因为**普通方法不受锁的影响**

在主方法中加入两个phone，

```java
package com.juc;

import java.util.concurrent.TimeUnit;

public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        Phone phone1 = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{
            //先获得锁，先执行
            phone1.sendMs();
        }, "A").start();

        //TimeUnit.SECONDS.sleep(1); //sleep 1秒

        new Thread(()->{
            try {
                phone2.call();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();

    }
}

//锁的是对象的调用者，在本例中，锁的是phone，谁先拿到谁先执行
class Phone {
    public synchronized void sendMs() {
        System.out.println("sendMs");
    }
    public synchronized void call() throws InterruptedException {
        TimeUnit.SECONDS.sleep(1);
        System.out.println("call");
    }
    public void hello () {
        System.out.println("hello");
    }
}
```

输出先为sendMs，后为call。这是因为锁的是调用方法的对象，而该例中有两个phone，互不影响，因此先调用无延迟的sendMs 

当为静态方法时，在一开始就被加载，synchronized锁的是Class，而不是对象

```java
package com.juc;

import java.util.concurrent.TimeUnit;

public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            //先获得锁，先执行
            phone.sendMs();
        }, "A").start();

        //TimeUnit.SECONDS.sleep(1); //sleep 1秒

        new Thread(()->{
            try {
                phone.call();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();

    }
}

//当为静态方法时，在一开始就被加载，synchronized锁的是Class，而不是对象
class Phone {
    public static synchronized void sendMs() {
        System.out.println("sendMs");
    }
    public static synchronized void call() throws InterruptedException {
        TimeUnit.SECONDS.sleep(1);
        System.out.println("call");
    }
    public void hello () {
        System.out.println("hello");
    }
}
```

输出结果为：sendMs     call

同样，当为两个phone时：

```java
package com.juc;

import java.util.concurrent.TimeUnit;

public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        //两个对象的Class相同，均为Phone
        Phone phone1 = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{
            //先获得锁，先执行
            phone1.sendMs();
        }, "A").start();

        //TimeUnit.SECONDS.sleep(1); //sleep 1秒

        new Thread(()->{
            try {
                phone2.call();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();

    }
}

//当为静态方法时，在一开始就被加载，synchronized锁的是Class，而不是对象
class Phone {
    public static synchronized void sendMs() {
        System.out.println("sendMs");
    }
    public static synchronized void call() throws InterruptedException {
        TimeUnit.SECONDS.sleep(1);
        System.out.println("call");
    }
    public void hello () {
        System.out.println("hello");
    }
}
```

输出结果为：sendMs     call

当改为普通同步方法和静态同步方法时，一个锁的为对象，一个锁的为Class

```java
package com.juc;

import java.util.concurrent.TimeUnit;

public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            //先获得锁，先执行
            phone.sendMs();
        }, "A").start();

        //TimeUnit.SECONDS.sleep(1); //sleep 1秒

        new Thread(()->{
            try {
                phone.call();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();

    }
}

//当为静态方法时，在一开始就被加载，synchronized锁的是Class，而不是对象
class Phone {
    //静态同步方法  锁的为Class
    public static synchronized void sendMs() {
        System.out.println("sendMs");
    }
    //改为普通同步方法   锁的为对象
    public synchronized void call() throws InterruptedException {
        TimeUnit.SECONDS.sleep(1);
        System.out.println("call");
    }
    public void hello () {
        System.out.println("hello");
    }
}
```

输出为：sendMs    call

当改为两个phone时

```java
package com.juc;

import java.util.concurrent.TimeUnit;

public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        Phone phone1 = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{
            //先获得锁，先执行
            phone1.sendMs();
        }, "A").start();

        //TimeUnit.SECONDS.sleep(1); //sleep 1秒

        new Thread(()->{
            try {
                phone2.call();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();

    }
}

//当为静态方法时，在一开始就被加载，synchronized锁的是Class，而不是对象
class Phone {
    //静态同步方法  锁的为Class
    public static synchronized void sendMs() {
        System.out.println("sendMs");
    }
    //改为普通同步方法   锁的为对象
    public synchronized void call() throws InterruptedException {
        TimeUnit.SECONDS.sleep(1);
        System.out.println("call");
    }
    public void hello () {
        System.out.println("hello");
    }
}
```

输出为 sendMs    call

小结：

- 静态同步方法锁的为Class
- 普通同步方法锁的为对象
- 同步方法和普通方法互不影响

# CopyOnWriteArrayList

java.util.ConcurrentModificationException 并发修改异常

修改方法：
1、List<String> list = new Vector<>(); Vector 1.0版本;ArrayList 1.2版本
2、List<String> list = Collections.synchronizedList(new ArrayList<>());  使用Collections中的方法
3、List<String> list = new CopyOnWriteArrayList<>();使用JUC中的CopyOnWriteArrayList类

根据[JMCui](https://www.cnblogs.com/jmcui/)博客[写入时复制（CopyOnWrite）](https://www.cnblogs.com/jmcui/p/12377081.html)

CopyOnWrite：写入时复制（简称COW），当多个对象指向相同资源，访问而不修改时，使用简单的无锁访问方法，当调用者修改对象时，系统复制一份后修改这个副本，然后将原数据替换为当前副本

CopyOnWrite优缺点：

- 优点：适合一些读多写少的数据。CopyOnWrite仅在增删改时加锁，为Lock锁，而Vector的增删改查方法都加入了synchronized来保证同步，因此CopyOnWrite效率更高
- 缺点：
  - 数据一致性问题，这种方法实现的是数据的最终一致性，当添加拷贝的数据没有进行替换时，读取到的仍未旧数据。
  - 内存占用问题，若对象较大，频繁调用会消耗内存

# CopyOnWriteArraySet

```java
package com.juc;

import java.util.HashSet;
import java.util.Set;

public class TestSet {
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();

        //java.util.ConcurrentModificationException
        for (int i = 0; i < 100; i++) {
            new Thread(()->{
                set.add(Thread.currentThread().getName());
                System.out.println(set);
            }).start();
        }
    }
}
```

对于并发修改异常：

1. Set<String> set = Collections.synchronizedSet(new HashSet<>());
2. Set<String> set = new CopyOnWriteArraySet<>();

# ConcurrentHashMap

```java
package com.juc;

import java.util.HashMap;
import java.util.Map;

public class TestMap {
    public static void main(String[] args) {
        
        Map<String, String> map = new HashMap<>();
        //java.util.ConcurrentModificationException
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(), "A");
                System.out.println(map);
            }).start();
        }

    }
}
```

解决方法：

1. Map<String, String> map = Collections.synchronizedMap(new HashMap<>());
2. Map<String, String> map = new Hashtable<>();在所有方法前加synchronized，效率较低
3. Map<String, String> map = new ConcurrentHashMap<>();

根据[xuefeng0707](https://blog.csdn.net/xuefeng0707)博客[HashMap与ConcurrentHashMap的区别](https://blog.csdn.net/xuefeng0707/article/details/40834595)

ConcurrentHashMap引入了一个“分段锁”的概念，具体可以理解为把一个大的Map拆分成N个小的HashTable，根据key.hashCode()来决定把key放到哪个HashTable中。

# Callable

Callable 接口类似于 Runnable，两者都是为那些其实例可能被另一个线程执行的类设计的。但是  Runnable 不会返回结果，并且无法抛出经过检查的异常

Callable接口“

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

泛型和方法返回值类型相同

**FutureTask为Runnable接口的一个实现类，FutureTask的构造方法中，参数为Callable**

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210313065242FutureTask.JPG)

FutureTask中的get方法，可获得Callable的返回值

```java
package com.juc;

import com.thread.state.TestJoin;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class TestCallable {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        TestCall testCall = new TestCall();
        FutureTask futureTask = new FutureTask(testCall);
        new Thread(futureTask).start();
        String tag = (String) futureTask.get(); //抛出异常
        System.out.println(tag);
    }
}

class TestCall implements Callable<String> {

    @Override
    public String call() throws Exception {
        System.out.println("ABC");
        return "123";
    }
}
```

当Call方法中较为耗时时，get方法可能会产生阻塞

# 常用辅助类

## CountDownLatch

同步辅助类，CountDownLatch为减法计数器

```java
package com.juc;

import java.util.concurrent.CountDownLatch;

//减法计数器
public class TestCountDown {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch count = new CountDownLatch(5); //参数为初始值
        for (int i = 0; i < 5; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName());
                count.countDown(); //-1
            }, String.valueOf(i)).start();
        }
        count.await(); //等待计数器归零后再向下执行
        System.out.println("end");
    }
}
```

- count.countDown();为计数器减一
- count.await();等到计数器值为0后才会向下执行

## CyclicBarrier

同步辅助类，加法计数器

构造方法：

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210313071923Cyc.JPG)

- await方法

当所有调用await方法的线程数量到达parties后，会执行Runnable线程中的run方法

```java
package com.juc;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

//加法计数器
public class TestCyclic {
    public static void main(String[] args) {
        CyclicBarrier cyc = new CyclicBarrier(5, ()->{
            System.out.println("end");
        });
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                try {
                    System.out.println(Thread.currentThread().getName());
                    cyc.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();

        }
    }
}
```

## Semaphore

```java
Semaphore.acquire(); //获得一个许可，若已满，则线程等待
Semaphore.release(); //释放一个许可，然后唤醒等待的线程
```

作用：**多个共享资源互斥，并发限流，控制最大的线程数**

```java
package com.juc;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

public class TestSemaphore {
    public static void main(String[] args) {
        //参数为最大线程数
        Semaphore semaphore = new Semaphore(3);

        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "acquire");
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                    System.out.println(Thread.currentThread().getName() + "release");
                }
            }).start();
        }
    }
}
```

# 读写锁

实现类：ReentrantReadWriteLock

接口 ReadWriteLock：维护一对相关锁，一个用于读，一个用于写。**写入锁为独占锁，读锁为共享锁**

```java
Lock readLock();//返回用于读取操作的锁。 
Lock writeLock();//返回用于写入操作的锁。 
readLock().lock();
readLock().unlock();
writeLock().lock();
writeLock().unlock();
```

```java
package com.juc;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class TestReadWriteLock {
    public static void main(String[] args) {
        Test4 test4 = new Test4();
        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(()->{
                test4.put(finalI + "", finalI + "");
                test4.get(finalI + "");
            }, String.valueOf(i)).start();
        }

    }
}

class Test4 {
    private Map<String, String> map = new HashMap<>();
    ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void put (String key, String value) {
        readWriteLock.writeLock().lock();

        try {
            System.out.println(Thread.currentThread().getName() + "put");
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "put完成");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }

    }
    public void get (String key) {
        readWriteLock.readLock().lock();

        try {
            System.out.println(Thread.currentThread().getName() + "get");
            map.get(key);
            System.out.println(Thread.currentThread().getName() + "get完成");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }

    }

}
```

# 阻塞队列BlockingDeque接口

队列：FIFO，先入先出原则。

当队列满时，则写入阻塞；当队列空时，则读取阻塞。

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210316123540Queue.JPG)

| 操作         | 抛出异常 | 不抛出异常 | 阻塞等待 | 超时等待 |
| ------------ | -------- | ---------- | -------- | -------- |
| 添加         | add      | offer      | put      | offer    |
| 移除         | remove   | poll       | take     | poll     |
| 检测队首元素 | element  | peek       | -        | -        |

```java
boolean add(E e);//添加元素，若容量已满，抛出异常java.lang.IllegalStateException: Queue full
boolean remove(Object o);//移除头元素，若队列为空，抛出异常java.util.NoSuchElementException
boolean offer(E e); //添加元素，若容量已满，返回false
E poll(); //移除头元素，若队列为空，返回null
E element(); //查询头元素，若为空，则抛出异常 java.util.NoSuchElementException
E element(); //查询头元素，若为空，则返回null
void put(E e); //添加一个元素，若已满，则阻塞等待
boolean offer(E e, long timeout, TimeUnit unit); //添加元素，并设置超时等待时间，unit为时间单位，timeout为时间数值，添加成功返回true，失败返回false
E take(); //移除头元素，若为空，则阻塞等待
E poll(long timeout, TimeUnit unit); //移除头元素，并设置超时等待时间
```

# SynchronousQueue

```java
public class SynchronousQueue<E>extends AbstractQueue<E>implements BlockingQueue<E>, Serializable
```

SynchronousQueue实现了BlockingDeque接口

同步队列，添加一个元素后必须取出来后才能再次存储下一个元素，进一个取一个

```java
package com.juc;

import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

public class TestSynchronousQueue {
    public static void main(String[] args) {
        SynchronousQueue synchronousQueue = new SynchronousQueue();

        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName() + " put 1");
                synchronousQueue.put(1);
                System.out.println(Thread.currentThread().getName() + " put 2");
                synchronousQueue.put(2);
                System.out.println(Thread.currentThread().getName() + " put 3");
                synchronousQueue.put(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "T1").start();
        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
                System.out.println(Thread.currentThread().getName() + " take->" + synchronousQueue.take());
                TimeUnit.SECONDS.sleep(1);
                System.out.println(Thread.currentThread().getName() + " take->" + synchronousQueue.take());
                TimeUnit.SECONDS.sleep(1);
                System.out.println(Thread.currentThread().getName() + " take->" + synchronousQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "T2").start();

    }
}
```

# 线程池

## 创建（三大方法）

池化技术：事先准备好一些资源，随用随拿，用完即还

优点：线程复用，控制最大线程数，管理线程

1. 降低资源消耗
2. 提高响应速度
3. 方便管理

Executors工具类，可用来创建线程池

```java
Executors.newSingleThreadExecutor(); //单个线程
Executors.newFixedThreadPool('3'); //容量为3的线程池
Executors.newCachedThreadPool(); //缓存线程池，容量可收缩
```

**以上本质上都是调用的ThreadPoolExecutor**

使用线程池开启线程：

```java
ExecutorService service = Executors.newCachedThreadPool();
service.execute(()->{
    //run()方法
}); //参数为Runnable接口
```

线程池使用结束后需要使用shutdown方法关闭

```java
package com.juc;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TestT {
    public static void main(String[] args) {
//        ExecutorService service = Executors.newSingleThreadExecutor(); //单个线程
//        ExecutorService service = Executors.newFixedThreadPool(10);
        ExecutorService service = Executors.newCachedThreadPool();

        try {
            for (int i = 0; i < 100; i++) {
                service.execute(()->{
                    System.out.println(Thread.currentThread().getName());
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //线程池使用结束后需要关闭
            service.shutdown();
        }


    }
}
```

## 源码分析（七大参数）

```java
//单个线程
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
//固定大小
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
//缓存线程池
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,   //MAX_VALUE=21亿
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
}
//可发现，以上本质上都是调用的ThreadPoolExecutor
//可发现，ThreadPoolExecutor方法有7个参数
public ThreadPoolExecutor(int corePoolSize,  //核心线程池大小，核心线程不会被关闭
                              int maximumPoolSize, //最大线程池大小
                              long keepAliveTime, //无调用下存活时间，关闭的是除了核心线程外的其他线程
                              TimeUnit unit,  //存活时间单位
                              BlockingQueue<Runnable> workQueue, //阻塞队列
                              ThreadFactory threadFactory, //线程工厂，用于创建线程，不用动
                              RejectedExecutionHandler handler //拒绝策略) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
}
```

![image-20210320154708308](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210320154708308.png)

## ThreadPoolExecutor

根据阿里巴巴Java开发手册：

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210320074515poll.JPG)



4大拒绝策略：

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210320075417%E6%8B%92%E7%BB%9D%E7%AD%96%E7%95%A5.png)

```java
new ThreadPoolExecutor.AbortPolicy();//默认拒绝策略，达到最大线程数后，若还有线程请求，则抛出异常java.util.concurrent.RejectedExecutionException
new ThreadPoolExecutor.CallerRunsPolicy(); //返回给上一个执行该内容的线程处理，即回到原线程
new ThreadPoolExecutor.DiscardPolicy(); //到达最大线程数后，丢弃任务，不会抛出异常
new ThreadPoolExecutor.DiscardOldestPolicy(); //到达最大线程后，尝试和最早线程竞争，不会抛出异常
```

所能运行的最大线程数：maximumPoolSize+阻塞队列长度

```java
package com.juc;

import java.util.concurrent.*;

public class TestThreadPoolExecutor {
    public static void main(String[] args) {
        //所能运行的最大线程数：maximumPoolSize+阻塞队列长度
        ExecutorService service = new ThreadPoolExecutor(
                2,
                3,
                2,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(2),
                Executors.defaultThreadFactory(), //默认线程线程工厂
                new ThreadPoolExecutor.AbortPolicy());

        try {
            for (int i = 0; i < 5; i++) {
                service.execute(()->{
                    System.out.println(Thread.currentThread().getName());
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            service.shutdown();
        }
    }
}
```

# CPU密集型和IO密集型

```java
System.out.println(Runtime.getRuntime().availableProcessors()); //获得CPU核数
```

线程池最大线程数：

CPU密集型线程池：将最大线程池数设置为 Runtime.getRuntime().availableProcessors()

IO密集型：最大线程池数和大型耗时任务线程数有关，如设置为两倍

# 四大函数式接口

函数式接口：只有一个方法的接口 @FunctionalInterface，均可用Lamda表达式进行简化

例如Runnable接口

根据[Turing·](https://blog.csdn.net/qq_41835813)的博客[Java8 四大函数式接口](https://blog.csdn.net/qq_41835813/article/details/106359068)，在java.util.function包中

![](https://img-blog.csdnimg.cn/20200526162719247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxODM1ODEz,size_16,color_FFFFFF,t_70)

## Function

函数型接口

```java
@FunctionalInterface
public interface Function<T, R> {
	R apply(T t);//传入参数为T，返回类型为R
} 
```

```java
package com.juc;

import java.util.function.Function;

public class TestFunction {
    public static void main(String[] args) {
        /*Function<String, String> function = new Function<String, String>() {
            @Override
            public String apply(String s) {
                return s;
            }
        };
        System.out.println(function.apply("A"));*/
        //使用Lamda表达式简化
        //工具类
        Function function = (s)->{return s;};
        System.out.println(function.apply(1));
    }
}
```

## Predicate

断定型接口，返回参数为boolean值

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t); 
}
```

```java
package com.juc;

import java.util.function.Predicate;

public class TestPredicate {
    public static void main(String[] args) {
        /*Predicate<String> predicate = new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.isEmpty(); //判断字符串是否为空
            }
        };
        System.out.println(predicate.test("a"));*/
        Predicate<String> predicate = (s)->{
            return s.isEmpty();
        };
        System.out.println(predicate.test("A"));

    }
}
```

## Consumer

消费型接口，方法只有输入没有返回值

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

```java
package com.juc;

import java.util.function.Consumer;

public class TestConsumer {
    public static void main(String[] args) {
        /*Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };
        consumer.accept("ABC");*/
        Consumer<String> consumer = (s)->{
            System.out.println(s);
        };
        consumer.accept("ABC");
    }
}
```

## Supplier

供给型接口，没有参数，只有返回值

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

```java
package com.juc;

import java.util.function.Supplier;

public class TestSupplier {
    public static void main(String[] args) {
        /*Supplier<String> supplier = new Supplier<String>() {
            @Override
            public String get() {
                return "ABC";
            }
        };
        System.out.println(supplier.get());*/
        Supplier<String> supplier = ()->{
            System.out.println("A");
            return "ABC";
        };
        System.out.println(supplier.get());
    }
}
```

# Stream流式计算

详细可参考[云深i不知处](https://blog.csdn.net/mu_wind)的博客[Java8 Stream：2万字20个实例，玩转集合的筛选、归约、分组、聚合](https://blog.csdn.net/mu_wind/article/details/109516995)

```java
package com.juc;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class TestStream {
    public static void main(String[] args) {
        Student s1 = new Student("A", 11, 13);
        Student s2 = new Student("A", 12, 12);
        Student s3 = new Student("A", 14, 13);

        List<Student> students = Arrays.asList(s1, s2, s3);
        //链式编程
        students.stream()   //传入参数的类型和List<Student>类型相同
                .filter((student)->{return student.getId()%2 == 0;})
                .filter((student)->{return student.getAge() > 12;})  //filter过滤，参数为断定型接口Predicate
                .map((student) -> {return student.getName().toLowerCase();}) //map映射，参数为函数式接口Function
                .sorted((u1, u2)->{return u2.compareTo(u1);}) //默认从小到大，参数为Comparator，也为一个函数式接口，可使用Lamda表达式
                .limit(1) //限制数量
                .forEach(System.out::println);
                /**
                 * Stream<T> sorted(Comparator<? super T> comparator);
                 */

class Student {
    private String name;
    private int id;
    private int age;

    public Student(String name, int id, int age) {
        this.name = name;
        this.id = id;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", id=" + id +
                ", age=" + age +
                '}';
    }
}

```

# ForkJoin

分支合并：并行执行任务，提高效率

特点：工作窃取，一个线程执行完后可执行其他线程未执行完的任务，使用双端队列

根据[华山拎壶冲](https://blog.csdn.net/tyrroo)的博客[Fork/Join框架基本使用](https://blog.csdn.net/tyrroo/article/details/81390202)

```java
package com.juc;

import java.util.concurrent.*;

public class TestForkJoin {
    private static final long MAX = 10000;

    static class MyForkJoin extends RecursiveTask<Long> {
        //开始时计算的值
        private long startValue;
        //结束时计算的值
        private long endValue;

        public MyForkJoin(long startValue, long endValue) {
            this.startValue = startValue;
            this.endValue = endValue;
        }

        @Override
        protected Long compute() {
            //普通for循环计算
            if (endValue - startValue < MAX) {
                //System.out.println("计算的部分：startValue = " + startValue + ", endValue = " + endValue);
                long totalValue = 0;
                for (long i = this.startValue; i <= this.endValue; i++) {
                    totalValue = totalValue + i;
                }
                return totalValue;
            }
            //拆分
            else {
                MyForkJoin task1 = new MyForkJoin(startValue, (startValue + endValue) / 2);
                task1.fork();
                MyForkJoin task2 = new MyForkJoin((startValue + endValue) / 2, endValue); //递归思想
                task2.fork();
                return task1.join() + task2.join();
            }

        }
    }

    public static void main(String[] args) {
        long startTime = System.currentTimeMillis();

        //使用ForkJoinPool开启
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = forkJoinPool.submit(new MyForkJoin(1, 10000_0000));
        try {
            long result = task.get();
            System.out.println("result = " + result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

        /*long total = 0;
        for (int i = 0; i <= 10000_0000; i++) {
            total = total + i;
        }
        System.out.println("total = " + total);*/

        long endTime = System.currentTimeMillis();
        System.out.println("所用时间：" + (endTime - startTime));
    }
}
```

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210321072330ForkJoin.jpg)

fork方法用于将新创建的子任务放入当前线程的work queue队列中，Fork/Join框架将根据当前正在并发执行ForkJoinTask任务的ForkJoinWorkerThread线程状态，决定是让这个任务在队列中等待，还是创建一个新的ForkJoinWorkerThread线程运行它，又或者是唤起其它正在等待任务的ForkJoinWorkerThread线程运行它。

join方法用于让当前线程阻塞，直到对应的子任务完成运行并返回执行结果。或者，如果这个子任务存在于当前线程的任务等待队列（work queue）中，则取出这个子任务进行“递归”执行。其目的是尽快得到当前子任务的运行结果，然后继续执行。

# 异步回调

```java
package com.juc;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

/**
 * 异步执行
 */
public class TestFuture {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //无返回值的异步回调
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("CompletableFuture");

        }); //参数为Runnable接口
        System.out.println("不是异步执行的内容"); //先执行这一部分，后获得异步执行的内容
        completableFuture.get();

        //有返回值的异步回调
        CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(()->{ //参数为供给型接口
            System.out.println("有返回值的异步回调 " + Thread.currentThread().getName());
            int a = 10/0; //编译失败
            return 1024; //编译成功的返回值
        });
        completableFuture2.whenComplete((t, u)->{
            System.out.println("t->" + t);
            System.out.println("u->" + u);
        }).exceptionally((e)->{
            System.out.println(e.getMessage());
            //编译失败的返回值
            return 10;
        }); //whenComplete当编译成功，参数为消费型接口(t, u)
            //exceptionally编译失败，参数为函数型接口，函数型接口里面为异常类
        //System.out.println(completableFuture2.get());
    }
}
```

# JMM和Volatile

Volatile：是Java虚拟机提供的**轻量级同步机制**

1. 保证可见性
2. 不保证原子性
3. 禁止指令重排

JMM：Java内存模型（并不存在）

关于JMM的一些同步约定：

1. 线程解锁前，必须把共享变量立即刷新入主内存
2. 线程加锁前，将主内存中最新数据读取到工作内存
3. 加锁和解锁是同一把锁

内存分为：**工作内存、主内存**

根据[牧竹子](https://blog.csdn.net/zjcjava)的博客[JMM概述](https://blog.csdn.net/zjcjava/article/details/78406330)

![](https://img-blog.csdnimg.cn/img_convert/8cedf683cdfacb3cfcd970cd739d5b9d.png)

## 保证可见性

存在的问题：

```java
package com.juc;

import java.util.concurrent.TimeUnit;

//存在的问题
public class TestJMM {
    static int temp = 0;
    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            while (temp == 0) {

            }
        }, "MyThread").start();

        TimeUnit.SECONDS.sleep(1);

        temp = 1;
        System.out.println(temp);
    }
}
```

temp=1后，MyThread线程并不会终止，即主内存中变量发生变化后，工作内存中的变量值并未改变，MyThread对主内存的值不可见

修改为：

```java
static volatile int temp = 0;
```

当temp=1后，MyThread立即停止

## 不保证原子性

```java
package com.juc;

public class TestJMM2 {
    //volatile不保证原子性
    private volatile static int sum = 0;
    public static void add() {
        sum++;
    }
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(()->{
                for (int j = 0; j < 100; j++) {
                    add();
                }
            }).start();
        }
        while (Thread.activeCount() > 2) { //main 和 gc线程一直存活
            Thread.yield();
        }
        System.out.println(sum);
    }
}
```

将add方法改为synchronized后，将保证原子性

sum++并不是原子性操作，分为

- 获取该值
- +1
- 返回该值

使用原子类比synchronized和lock高效很多

```java
package com.juc;

import java.util.concurrent.atomic.AtomicInteger;

public class TestJMM2 {
    //volatile不保证原子性
    //使用原子类
    private volatile static AtomicInteger sum = new AtomicInteger(); //参数为原始值，默认为0
    public static void add() {
        sum.getAndIncrement(); //原子类+1操作
    }
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(()->{
                for (int j = 0; j < 100; j++) {
                    add();
                }
            }).start();
        }
        while (Thread.activeCount() > 2) { //main 和 gc线程一直存活
            Thread.yield();
        }
        System.out.println(sum);
    }
}
```

## 指令重排

计算机不一定按照所写的顺序执行

源代码-->编译器优化重排-->指令并行重排-->内存系统重排-->执行

处理器在指令重排时，会考虑数据的依赖性

使用volatile可避免指令重排：

volatile使用内存屏障，禁止顺序执行顺序变换

1. 保证特定的操作执行顺序
2. 保证某些变量的内存可见性（利用的是volatile的可见性）

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1929836/o_210321090649%E7%A6%81%E6%AD%A2%E9%87%8D%E6%8E%92.jpg)

# 单例模式

构造器私有

饿汉式单例：在一开始便被加载，浪费内存空间

```java
package com.juc;

public class Hungry {
    //构造器私有
    private Hungry () {

    }

    Byte[] test1 = new Byte[1024];
    Byte[] test2 = new Byte[1024];
    Byte[] test3 = new Byte[1024];
    //一开始就被加载，可能会导致浪费空间内存
    private static final Hungry hungry = new Hungry();

    public static Hungry getInstance() {
        return hungry;
    }

}
```

懒汉式单例，可用反射破坏单例模式

```java
package com.juc;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;

public class Lazy {

    private static boolean flag = false;

    //构造器私有
    private Lazy () {
        synchronized (Lazy.class) {
            if (flag == false) {
                flag = true;
            } else {
                throw new RuntimeException("反射破坏单例");
            }
        }

        System.out.println(Thread.currentThread().getName());
    }

    private volatile static Lazy lazy;

    //双重检测锁模式，DCL懒汉式单例
    public static Lazy getInstance() {
        //需要时再new
        //加锁
        if (lazy == null) {
            synchronized (Lazy.class) {
                if (lazy == null) {
                    lazy = new Lazy(); //不是原子性才做
                    /**
                     * 1、创建内存空间
                     * 2、执行构造方法，初始化对象
                     * 3、将对象指向内存空间
                     * 由于指令重排，可能在多线程中会造成一些问题，例如执行132操作,下一个线程B可能会直接执行return lazy
                     * 因此需要加volatile
                     */
                }
            }
        }

        return lazy;
    }

    //使用反射破坏单例
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchFieldException {
        //Lazy instance = Lazy.getInstance();

        Field flag = Lazy.class.getDeclaredField("flag");
        flag.setAccessible(true);

        Constructor<Lazy> declaredConstructor = Lazy.class.getDeclaredConstructor(null);
        //declaredConstructor.setAccessible(true);
        Lazy instance = declaredConstructor.newInstance();
        flag.set(instance, false); //3、使用反射对标志位进行修改
        Lazy instance1 = declaredConstructor.newInstance(); //2、用反射创建两个对象
        System.out.println(instance);
        System.out.println(instance1); //1、反射破坏单例

    }

}
```

# CAS

AtomicInteger中的getAndIncrement方法

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
   		//此为自旋锁
        do {
            //获取var1下var2内存地址偏移量的值
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
		//compareAndSwapInt方法，若（var1, var2）值为var5，则var5 + var4，var4=1，即var5+1
        return var5;
}
```

```java
package com.juc;

import java.util.concurrent.atomic.AtomicInteger;

public class TestCAS {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(10);
        //若达到except值，则更新为update值
        System.out.println(atomicInteger.compareAndSet(10, 11));
        System.out.println(atomicInteger.get());
    }
}
```

Java通过调用C++（native）或Unsafe来调用内存

CAS：比较当前内存和主内存中的值，若相等，则更新，否则一直循环

缺点：

1. 循环浪费时间
2. 一次性保证一个共享变量的原子性
3. 存在ABA问题

# ABA问题

ABA问题的根本在于cas在修改变量的时候，无法记录变量的状态，比如修改的次数，否修改过这个变量。这样就很容易在一个线程将A修改成B时，另一个线程又会把B修改成A,造成cas多次执行的问题。

```java
package com.juc;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicStampedReference;

public class TestABA {
    public static void main(String[] args) {
        //第一个参数为初始值，第二个参数为版本号，每次被修改，版本号+1
        AtomicStampedReference<Integer> integerAtomicStampedReference = new AtomicStampedReference<>(10, 1);

        new Thread(()->{

            int stamp = integerAtomicStampedReference.getStamp();
            System.out.println(stamp);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //后面两个参数为预期stamp和新的stamp
            System.out.println(integerAtomicStampedReference.compareAndSet(10, 20,
                              integerAtomicStampedReference.getStamp(), integerAtomicStampedReference.getStamp() + 1));
            System.out.println(integerAtomicStampedReference.getStamp());
        }, "A").start();
        new Thread(()->{

            int stamp = integerAtomicStampedReference.getStamp();
            System.out.println(stamp);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(integerAtomicStampedReference.compareAndSet(10, 15,
                                                                            stamp, stamp + 1));
        }, "B").start();


    }
}
```

# 可重入锁

**公平锁**：线程必须排队

**非公平锁**：线程可以插队（默认）

```java
//默认非公平锁
public ReentrantLock() {
    sync = new NonfairSync();
}
//当创建时加入参数true，则为公平锁
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
}
```

使用synchronized锁

```java
package com.juc;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Demo01 {
    public static void main(String[] args) {
        TestDemo01 t = new TestDemo01();

        new Thread(()->{
            t.m1();
        },"A").start();
        new Thread(()->{
            t.m1();
        },"B").start();
    }
}

class TestDemo01 {
    public synchronized void m1() {
        System.out.println(Thread.currentThread().getName() + "m1");
        m2();
    }
    public synchronized void m2() {
        System.out.println(Thread.currentThread().getName() + "m2");
    }
}
```

结果为：

Am1
Am2
Bm1
Bm2

使用Lock锁：

```java
package com.juc;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Demo02 {
    public static void main(String[] args) {
        TestDemo02 t = new TestDemo02();

        new Thread(()->{
            t.m1();
        },"A").start();
        new Thread(()->{
            t.m1();
        },"B").start();
    }
}

class TestDemo02 {
    Lock lock = new ReentrantLock();
    public void m1() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "m1");
            m2();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }
    public void m2() {
        Lock lock = new ReentrantLock();
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "m2");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

结果为

Am1
Am2
Bm1
Bm2

注意Lock锁和synchronized锁的区别，Lock锁的lock和unlock必须配对，当出现

```java
lock.lock();
lock.lock();
xxx
lock.unlock();
```

这时，线程将会被一直锁在里面，无法继续进行

# 自旋锁

```java
do {
            //获取var1下var2内存地址偏移量的值
            var5 = this.getIntVolatile(var1, var2);
} while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
```

使用CAS完成自旋锁

```java
package com.juc;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

public class SpinLock {
    public static void main(String[] args) {
        MyLock myLock = new MyLock();

        new Thread(()->{
            myLock.lock();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                myLock.unLock();
            }
        }, "T1").start();

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            myLock.lock();
            //必须等到unLock方法中的期望为T2线程时才会解锁
            myLock.unLock();

        }, "T2").start();


    }
}

class MyLock {
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    //加锁
    public void lock () {
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() + "->lock()");

        //自旋锁
        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }
    //解锁
    public void unLock () {
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() + "->unLock()");

        atomicReference.compareAndSet(thread, null);
    }

}
```

# 死锁

多个进程在运行过程中因争夺资源而造成的一种僵局，当进程处于这种僵持状态时，若无外力作用，它们都将无法再向前推进。

![死锁](E:\java截图\死锁.jpg)

排查死锁：

1. jps-l，查看进程号

   ![死锁截图](E:\java截图\死锁截图.jpg)

2. jstack +线程号，查看线程信息

![image-20210324195901804](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210324195901804.png)

```java
package com.juc;

import java.util.concurrent.TimeUnit;

public class Demo03 {
    public static void main(String[] args) {

        //字符串常量存储在堆中字符串常量池中，因此可造成死锁现象
        new Thread(new LockDemo3("A", "B"), "T1").start();
        new Thread(new LockDemo3("B", "A"), "T2").start();

    }
}

class LockDemo3 implements Runnable{
    private String A;
    private String B;

    public LockDemo3(String a, String b) {
        A = a;
        B = b;
    }

    @Override
    public void run() {
        synchronized (A) {
            System.out.println(Thread.currentThread().getName() + "get" + A + "->wants" + B);

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (B) {
                System.out.println(Thread.currentThread().getName() + "get" + B + "->wants" + A);
            }
        }
    }
}
```











====