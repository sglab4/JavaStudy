参考 https://zhuanlan.zhihu.com/p/271783423

模拟一个高占用 cpu 的场景

```java
public static void main(String[] args)  {
  for (int i = 0; i < 10; i++) {
    Thread thread = new Thread(() -> {
      System.out.println(Thread.currentThread().getName());
      try {
        Thread.sleep(30 * 60 * 1000);
      }catch (Exception e){
        e.printStackTrace();
      }
    });
    thread.setName("thread-" + i);
    thread.start();
  }

  Thread highCpuThread = new Thread(() -> {
    int i = 0;
    while (true) {
      i++;
    }
  });
  highCpuThread.setName("HighCpu");
  highCpuThread.start();
}
```

**第一步**，在 linux 系统中使用 top 找到占用 CPU 最高的 Java **进程**

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204071422317.jpeg)

**第二步**，用 `top -Hp` 命令查看占用 CPU 最高的**线程**

执行`top -Hp pid`命令，pid 就是前面的 Java 进程，我这个例子中就是 `13731` ，完整命令为：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204071423879.jpeg)

然后将 `13756`转换为 16 进制的

**第三步**，保存线程栈信息

当前 Java 程序的所有线程信息都可以通过 `jstack`命令查看，我们用`jstack`命令将第一步找到的 Java 进程的线程栈保存下来。

```text
jstack 13731 > thread_stack.log
```

**第四步**，在线程栈中查找罪魁祸首的线程

第二步已经找到了这个罪魁祸首的线程 PID，并把它转换成了 16 进制的，第三步保存下来的线程栈中有所有线程的 PID 16 进制信息，我们在线程栈中查找这个16进制的线程 id （`0x35bc`）。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204071424952.png)

