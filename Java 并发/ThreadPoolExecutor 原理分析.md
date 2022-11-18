参考 https://www.cnblogs.com/yszzu/p/10122658.html

执行 ThreadPoolExecutor 的 execute 方法时

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208071527519.png)

1、判断当前线程数是否小于corePoolSize, 若是，则调用 addWorker 方法创建新的核心worker对象

2、判断是否可将当前任务添加到阻塞队列中，若是，则添加

3、判断是否可开启非核心线程执行任务，若是，则执行，否则执行拒绝策略

addWorke() 方法如下

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208071530752.png)

线程被封装为 Work 对象执行任务，线程启动后，又做了哪些工作：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208071531974.png)

发现，本线程执行完当前任务后，会不断从阻塞队列中拿取任务，若执行过程中抛出了异常，则会执行 processWorkerExit 方法

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208071533996.png)

可以看出，有异常时旧的线程会被删除（GC回收），再创建新的Worker， 并标记为非核心线程。

