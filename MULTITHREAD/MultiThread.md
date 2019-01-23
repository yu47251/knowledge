# 多线程

## 常用的几种线程池
### 单线程池
```java
// 创建方式：
Executors.newSingleThreadExecutor();
// 实现过程：固定核心线程数和最大线程数为1，
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```
### 计划线程池
```java
// 创建方式：        
ScheduledExecutorService ses = Executors.newScheduledThreadPool(6);
// 5秒以后执行
ses.schedule(new BizThread(), 5000, TimeUnit.MILLISECONDS);
// 创建过程：
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
//ScheduledThreadPoolExecutor 继承 ThreadPoolExecutor
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
            new DelayedWorkQueue());
}
```
### 缓存线程池
- 该线程池的队列（SynchronousQueue）中不存放任何数据，只做同步功能。所以，当有任务需要执行的时候，通过同步功能以后，直接开辟核心线程数（0）以外的线程
```java
// 创建方式：
Executors.newCachedThreadPool();
// 设置核心线程数为0，最大线程数为int最大值：2147483647
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
}
```
### 固定线程池
```java
// 创建方式：
Executors.newFixedThreadPool(6);
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
}
```

- 总结： 以上代码可以看出，所有的类型的线程池都是用ThreadPoolExecuter创建，只是传参的不同。

## 线程池 ThreadPoolExecutor

### 构造函数
```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit, BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit, BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit, BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit, BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
}
```
- 以上四种构造器，前三种最终都是调用的第四种。 只是设置了默认的ThreadFactory和RejectedExecutionHandler

- ThreadFactory：线程工厂， 创建线程，源码中使用的默认线程工厂为：Executors.DefaultThreadFactory
```java
this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, Executors.defaultThreadFactory(), defaultHandler);
```
- RejectedExecutionHandler：表示当拒绝处理任务时的策略, 有以下四种取值：
```java
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 (默认值)
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 
```
## 线程安全
### volatile 关键字
#### volatile原理
