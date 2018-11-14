# Android 线程池

1. ThreadPoolExecutor 


Handler发送Message到MessageQueue中保存，Looper会不断的从MessageQueue中获取Message，然后调用与Message绑定Handler的dispatchMessage方法，最终调用会handleMessage进行真正消息处理

Executor作为一个接口，它的具体实现就是ThreadPoolExecutor。

Android中的线程池都是直接或间接通过配置ThreadPoolExecutor来实现不同特性的线程池。

```java
/* 
*@ ThreadPoolExecutor构造参数介绍 
*/  
public ThreadPoolExecutor(  
//核心线程数，除非allowCoreThreadTimeOut被设置为true，否则它闲着也不会死  
int corePoolSize,   
//最大线程数，活动线程数量超过它，后续任务就会排队                     
int maximumPoolSize,   
//超时时长，作用于非核心线程（allowCoreThreadTimeOut被设置为true时也会同时作用于核心线程），闲置超时便被回收             
long keepAliveTime,                            
//枚举类型，设置keepAliveTime的单位，有TimeUnit.MILLISECONDS（ms）、TimeUnit. SECONDS（s）等  
TimeUnit unit,  
//缓冲任务队列，线程池的execute方法会将Runnable对象存储起来  
BlockingQueue<Runnable> workQueue,  
//线程工厂接口，只有一个new Thread(Runnable r)方法，可为线程池创建新线程  
ThreadFactory threadFactory)

```



    1.execute一个线程之后，如果线程池中的线程数未达到核心线程数，则会立马启用一个核心线程去执行。

    2.execute一个线程之后，如果线程池中的线程数已经达到核心线程数，且workQueue未满，则将新线程放入workQueue中等待执行。

    3.execute一个线程之后，如果线程池中的线程数已经达到核心线程数但未超过非核心线程数，且workQueue已满，则开启一个非核心线程来执行任务。

    4.execute一个线程之后，如果线程池中的线程数已经超过非核心线程数，则拒绝执行该任务，采取饱和策略，并抛出RejectedExecutionException异常。

    队列会满，此时会开启2个非核心线程来执行剩下的两个任务。


2. FixedThreadPool (可重用固定线程数)

```java
public static ExecutorService newFixThreadPool(int nThreads){  
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());  
}  
//使用  
Executors.newFixThreadPool(5).execute(r);
```

特点：参数为核心线程数，只有核心线程，无非核心线程，并且阻塞队列无界。

FixThreadPool其实就像一堆人排队上公厕一样，可以无数多人排队，但是厕所位置就那么多，而且没人上时，厕所也不会被拆迁,由于线程不会回收，FixThreadPool会更快地响应外界请求


3. SingleThreadPool(单个核线的fixed)

```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

    //创建
    ExecutorService singleThreadPool = Executors.newSingleThreadExecutor();

```
由于只有一个核心线程，当被占用时，其他的任务需要进入队列等待。
可以把SingleThreadPool简单的理解为FixThreadPool的参数被手动设置为1的情况，即Executors.newFixThreadPool(1).execute(r)。

4.  CachedThreadPoo(按需创建)
```java

  public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }


     ExecutorService singleThreadPool = Executors.newCachedThreadPool();
```
没有核心线程，只有非核心线程，并且每个非核心线程空闲等待的时间为60s，采用SynchronousQueue队列
CachedThreadPool只有非核心线程，最大线程数非常大，所有线程都活动时，会为新任务创建新线程，否则利用空闲线程（60s空闲时间，过了就会被回收，所以线程池中有0个线程的可能）处理任务
任务队列SynchronousQueue相当于一个空集合，导致任何任务都会被立即执行

比较适合执行大量的耗时较少的任务。喝咖啡人挺多的，喝的时间也不长。


5. ScheduledThreadPool(定时延时执行)
   ```java
       public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }

    DEFAULT_KEEPALIVE_MILLIS =10 

    //使用，延迟1秒执行，每隔2秒执行一次Runnable r  
     Executors. newScheduledThreadPool (5).scheduleAtFixedRate(r, 1000, 2000, TimeUnit.MILLISECONDS);
   ```
核心线程数固定，非核心线程（闲着没活干会被立即回收）数没有限制。

从上面代码也可以看出，ScheduledThreadPool主要用于执行定时任务以及有固定周期的重复任务。

   
   
```java
    ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
    LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
    PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
    DelayQueue：一个使用优先级队列实现的无界阻塞队列。
    SynchronousQueue：一个不存储元素的阻塞队列。
    LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
    LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。
```
```java

1.shutDown()  关闭线程池，不影响已经提交的任务

2.shutDownNow() 关闭线程池，并尝试去终止正在执行的线程

3.allowCoreThreadTimeOut(boolean value) 允许核心线程闲置超时时被回收

4.submit 一般情况下我们使用execute来提交任务，但是有时候可能也会用到submit，使用submit的好处是submit有返回值。

5.beforeExecute() - 任务执行前执行的方法

6.afterExecute() -任务执行结束后执行的方法

7.terminated() -线程池关闭后执行的方法
```

https://blog.csdn.net/l540675759/article/details/62230562

https://www.jianshu.com/p/7b2da1d94b42