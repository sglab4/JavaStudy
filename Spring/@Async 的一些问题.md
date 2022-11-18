参考 https://developer.aliyun.com/article/768513

使用 @Async 使用默认线程池 SimpleAsyncTaskExecutor 执行任务

```java
protected Executor getDefaultExecutor(@Nullable BeanFactory beanFactory) {
   Executor defaultExecutor = super.getDefaultExecutor(beanFactory);
   return (defaultExecutor != null ? defaultExecutor : new SimpleAsyncTaskExecutor());
}
```

可以看到，它默认使用的线程池是`SimpleAsyncTaskExecutor`。我们不看这个类的源码，只看它上面的文档注释，如下：

![image-20200720160047340](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202203221027576.png)

主要说了三点

1. 为每个任务新起一个线程
2. 默认线程数不做限制
3. 不复用线程

由于Spring Boot默认用于异步任务的线程池是这样配置的：

![pasted-565.png](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202203221028309.webp) 

图中的两个重要参数是需要关注的：

- `queueCapacity`：缓冲队列的容量，默认为INT的最大值（2的31次方-1）。
- `maxSize`：允许的最大线程数，默认为INT的最大值（2的31次方-1）。

