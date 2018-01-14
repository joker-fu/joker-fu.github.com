---
title: Android的线程池与队列
copyright: true
date: 2018-01-02 22:37:22
categories: Android
tags: Android
---
Android中的线程池来源于Java中的Executor接口，真正的线程池实现为ThreadPoolExecutor，它提供参数来配置线程池。既然提到线程池，首先得了解线程池有什么有点，线程池的优点主要有以下3点：
1. 线程重用，通过重用线程池中线程避免反复创建和销毁县城带来的性能开销问题。
2. 控制并发数，避免大量线程同时工作抢占系统资源造成阻塞。
3. 简单的线程管理，实现定时执行等功能。

#### 一 ThreadPoolExecutor
1) __ThreadPoolExecutor 构造器__
```
     //使用给定的参数和默认线程工厂、拒绝执行的Handler创建一个新的ThreadPoolExecutor 
     ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue)
    
     //使用给定的参数和拒绝执行的Handler创建一个新的ThreadPoolExecutor。
     ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory)
     
     //使用给定的参数和默认线程工厂创建一个新的ThreadPoolExecutor
     ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler)

     //使用给定的参数创建一个新的ThreadPoolExecutor
     ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
 
```

2) __ThreadPoolExecutor 构造器参数说明__
- corePoolSize 核心线程数（常驻线程数）
一直保持在线程池的线程数，空闲状态也不会退出，除非设置allowCoreThreadTimeOut为true
- maximumPoolSize 最大线程数
线程池中允许存在的最大线程数
- keepAliveTime 保持活跃时间
当线程数大于核心线程数时，这是超出空闲线程在终止之前等待新任务的最大时间。当allowCoreThreadTimeOut为true时，也适用与核心线程
- unit 单位
keepAliveTime参数的时间单位
- workQueue 任务队列
在执行任务之前用于保存任务的队列。 该队列将仅保存由（Runnable的execute）方法提交的任务。
- threadFactory 线程工厂
executor创建新线程的时候使用
- handler 拒绝执行的handler
当线程池无法执行新任务，导致执行被阻止时使用的处理程序，可能因为线程达到线程限制和队列容量

3) __ThreadPoolExecutor执行任务时大致规则__ 
-  线程池中线程数小于核心线程数，会直接启动一个核心线程来执行新任务
-  线程池中线程数大于等于核心线程数， 将会将新任务存放到任务队列等待执行
- 线程池中线程数大于等于核心线程数，且任务队列已满，但线程池中线程数小于最大线程数，将会启动一个非核心线程来执行任务
- 线程池中线程数大于等于最大线程数，那么就会拒绝执行该任务，调用handler的rejectedException来通知调用者

4) __ThreadPoolExecutor在AsyncTask中的使用__ 
```
    private static final String LOG_TAG = "AsyncTask";

    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final int KEEP_ALIVE = 1;

    private static final ThreadFactory sThreadFactory = new ThreadFactory() {
        private final AtomicInteger mCount = new AtomicInteger(1);

        public Thread newThread(Runnable r) {
            return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
        }
    };

    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

    /**
     * An {@link Executor} that can be used to execute tasks in parallel.
     */
    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
```
通过源码，我们可以看到AsyncTask的THREAD_POOL_EXECUTOR属性配置如下：
- 核心线程数（CORE_POOL_SIZE）为CPU核心数+1
- 最大线程数（MAXIMUM_POOL_SIZE）为CPU核心数 * 2 + 1
- 非核心线程保持活跃时间为1，单位为秒
- 任务队列容量为128

以上对ThreadPoolExecutor和AsyncTask中的ThreadPoolExecutor配置进行了介绍。在Android中通过对ThreadPoolExecutor的配置实现了四类不同功能特性的线程池，接下来我们就对Android中的这四类线程池进行一个简单介绍

#### 二 FixedThreadPool
通过Executors.newFixedThreadPool创建一个线程池，它使用固定数量的线程操作了共享无界队列。在任何时候，大多数线程都是主动处理任务的。如果在所有线程处于活动状态时提交其他任务，则它们将在队列中等待，直到有线程空闲可用为止。如果任何线程在关闭前在执行过程中失败，如果需要执行后续任务，则新线程将取代它。线程池中线程会一致存在，直到线程池明确关闭（shutdown）。下面是它的实现，可以看到它只有不会被回收的核心线程，队列大小也没有限制。
```
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

#### 三 CachedThreadPool
通过Executors.newCachedThreadPool创建一个线程池，根据需要创建新线程，但在可用线程时将重用以前构建的线程。这些池通常会提高执行许多短期异步任务的程序的性能。如果先前创建线程可用的话，调用将重用先前构建的线程。如果没有现有的线程可用，一个新线程将被创建并添加到池。未使用六十秒的线程被终止并从缓存中移除。因此，空闲时间足够长的池不会消耗任何系统资源。下面是它的实现，可以看到与ThreadPoolExecutor不同的是它没有核心线程，最大线程数为Integer.MAX_VALUE，且CachedThreadPool的任务队列是一个SynchronousQueue的空集合，这将导致任务会被立即执行，所以这类线程比较适合执行大量耗时较少的任务。
```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

#### 四 ScheduledThreadPool
通过Executors.newScheduledThreadPoo创建一个线程池，可以在给定延迟后调度命令运行，或定期执行命令。它有数量固定的核心线程，且有数量无限多的非核心线程，但是它的非核心线程超时时间是0s，所以非核心线程一旦空闲立马就会被回收。这类线程池适合用于执行定时任务和固定周期的重复任务。
```
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }
```

#### 五 newSingleThreadScheduledExecutor
通过Executors.newSingleThreadScheduledExecutor创建一个单线程执行器，可以在给定延迟后调度命令运行，或定期执行命令。任务是按顺序执行的，在任何给定的时间内都不会有一个任务处于活动状态，让调用者可以忽略线程同步问题。
```
    public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
        return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1));
    }
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }
```
#### 六 线程池一般用法
- shutDown()，关闭线程池，需要执行完已提交的任务
- shutDownNow()，关闭线程池，并尝试结束已提交的任务
- allowCoreThreadTimeOut(boolen)，允许核心线程闲置超时回收
- execute()，提交任务无返回值
- submit()，提交任务有返回值

除了上面4种线程池，还可以根据实际需求自定义线程池。
#### 七 自定义线程池
```
ExecutorService mExecutor = Executors.newFixedThreadPool(5);
```
execute()方法，接收一个Runnable对象作为参数，异步执行。
```
Runnable myRunnable = new Runnable() {
    @Override
    public void run() {
        Log.i("myRunnable", "run");
    }
};
mExecutor.execute(myRunnable);
```

#### 八 队列简述

Queue的成员函数

      add        增加一个元索                     如果队列已满，则抛出一个IIIegaISlabEepeplian异常
      remove   移除并返回队列头部的元素    如果队列为空，则抛出一个NoSuchElementException异常
      element  返回队列头部的元素             如果队列为空，则抛出一个NoSuchElementException异常
      offer       添加一个元素并返回true       如果队列已满，则返回false
      poll         移除并返问队列头部的元素    如果队列为空，则返回null
      peek       返回队列头部的元素             如果队列为空，则返回null
      put         添加一个元素                      如果队列满，则阻塞
      take        移除并返回队列头部的元素     如果队列为空，则阻塞


remove、element、offer 、poll、peek 其实是属于Queue接口。

LinkedBlockingQueue是一个链表实现的阻塞队列，在链表一头加入元素，如果队列满，就会阻塞，另一头取出元素，如果队列为空，就会阻塞。

LinkedBlockingQueue内部使用ReentrantLock实现插入锁(putLock)和取出锁(takeLock)。putLock上的条件变量是notFull，即可以用notFull唤醒阻塞在putLock上的线程。takeLock上的条件变量是notEmtpy，即可用notEmpty唤醒阻塞在takeLock上的线程。

知道了LinkedBlockingQueue，再来理解ArrayBlockingQueue就比较好理解了。类似LinkList和ArrayList的区别。如果知道队列的大小，那么使用ArrayBlockIngQueue就比较合适了，因为它使用循环数组实现，但是如果不知道队列未来的大小，那么使用ArrayBlockingQueue就必然会导致数组的来回复制，降低效率。
