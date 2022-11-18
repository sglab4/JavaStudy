```java
public class Test {
    public static void main(String[] args) {
        //获取线程管理 MXBean
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        //不需要获取同步的 monitor 和 synchronize 信息，仅获取线程和线程堆栈信息
        ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
        //将线程 ID 和线程名称输出
        for (ThreadInfo threadInfo : threadInfos) {
            System.out.println(threadInfo.getThreadId() + "  " + threadInfo.getThreadName());
        }
    }
}
```

查看运行的线程

```html
6  Monitor Ctrl-Break
5  Attach Listener
4  Signal Dispatcher
3  Finalizer
2  Reference Handler
1  main
```

以上线程解释如下：参考 https://www.cnblogs.com/thisiswhy/p/14545623.html

**1、Reference Handler 线程：**

JVM 在创建 main 线程后就创建 Reference Handler 线程，其优先级最高，为 10，它主要用于处理引用对象本身（软引用、弱引用、虚引用）的垃圾回收问题。

**2、Finalizer 线程：**

这个线程也是在 main 线程之后创建的，其优先级为10，主要用于在垃圾收集前，调用对象的 finalize() 方法。
关于 Finalizer 线程的几点：

1)只有当开始一轮垃圾收集时，才会开始调用 finalize() 方法；因此并不是所有对象的 finalize() 方法都会被执行；

2)该线程也是 daemon 线程，因此如果虚拟机中没有其他非 daemon 线程，不管该线程有没有执行完 finalize() 方法，JVM 也会退出；

3) JVM在垃圾收集时会将失去引用的对象包装成 Finalizer 对象（Reference的实现），并放入 ReferenceQueue，由 Finalizer 线程来处理；最后将该 Finalizer 对象的引用置为 null，由垃圾收集器来回收；

4) JVM 为什么要单独用一个线程来执行 finalize() 方法呢？如果 JVM 的垃圾收集线程自己来做，很有可能由于在 finalize() 方法中误操作导致 GC 线程停止或不可控，这对 GC 线程来说是一种灾难。

**3、Attach Listener 线程：**

Attach Listener 线程是负责接收到外部的命令，而对该命令进行执行的并且把结果返回给发送者。通常我们会用一些命令去要求 jvm 给我们一些反馈信息。

如：java -version、jmap、jstack 等等。如果该线程在 jvm 启动的时候没有初始化，那么，则会在用户第一次执行 jvm 命令时，得到启动。

**4、Signal Dispatcher 线程：**

前面我们提到第一个 Attach Listener 线程的职责是接收外部 jvm 命令，当命令接收成功后，会交给 signal dispather 线程去进行分发到各个不同的模块处理命令，并且返回处理结果。signal dispather 线程也是在第一次接收外部 jvm 命令时，进行初始化工作。

**5、Monitor Ctrl-Break 线程：**

属于 IDEA 特有线程，来自 idea_rt 这个 jar 包中 MANIFEST.MF 文件的 AppMainV2.class 里面的 startMonitor 方法

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210908203605.png)

















