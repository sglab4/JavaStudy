参考 https://www.jianshu.com/p/49b9d15456d9

@Async 配置有两个，一个是执行的线程池，一个是异常处理

执行的线程池默认情况下找唯一的 org.springframework.core.task.TaskExecutor，或者一个 Bean 的 Name 为 taskExecutor 的 java.util.concurrent.Executor 作为执行任务的线程池。如果都没有的话，会创建 SimpleAsyncTaskExecutor 线程池来处理异步方法调用，当然 @Async 注解支持一个 String 参数，来指定一个 Bean 的 Name，类型是 Executor 或 TaskExecutor，表示使用这个指定的线程池来执行这个异步任务

异常处理，@Async 标记的方法只能是 void 或者 Future 返回值，在无返回值的异步调用中，异步处理抛出异常，默认是SimpleAsyncUncaughtExceptionHandler 的 handleUncaughtException() 会捕获指定异常，只是简单的输出了错误日志(一般需要自定义配置异常处理)，原有任务还会继续运行，直到结束(具有 void 返回类型的方法不能将任何异常发送回调用者，默认情况下此类未捕获异常只会输出错误日志)，而在有返回值的异步调用中，异步处理抛出了异常，会直接返回主线程处理，异步任务结束执行，主线程也会被异步方法中的异常中断结束执行

**@Async有两个使用的限制**

1. 它必须仅适用于 public 方法
2. 在同一个类中调用异步方法将无法正常工作(self-invocation)

