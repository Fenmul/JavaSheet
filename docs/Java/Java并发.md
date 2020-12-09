# 0 并发编程综述

## 0.1 单例模式

```java
public class Singleton {
  volatile static Singleton instance;
  // 也可以使用 final 修饰 1.5 版本以前会存在问题
  static Singleton getInstance(){
    if (instance == null) {
      synchronized(Singleton.class) {
        if (instance == null)
          instance = new Singleton();
        }
    }
    return instance;
  }
}
```


# 1 线程基础

## 1.1 线程的创建


线程的创建本质上只有一种就是**构造 Thread 类**，线程最终执行的都是 run 方法，区别在于运行内容的来源不同，重写 Runnable 就是来源于 target，继承就是来源于重写的 run 方法。


具体的实现可以有以下几种：
### 1.1.1 继承 Thread 类


```java
public class ExtendsThread extends Thread {
    @Override
    public void run() {
        System.out.println('用Thread类实现线程');
    }
}
```


run() 方法在 Thread 类中的具体实现如下：


```java
/* What will be run. */
private Runnable target;
public void run() {
    if (target != null) {
        target.run();
    }
}
```


因为重写了 run 方法所以直接执行 ExtendsThread 类的 run 方法。
### 1.1.2 实现 Runnable 接口


```java
public class RunnableThread implements Runnable {
    @Override
    public void run() {
        System.out.println('用实现Runnable接口实现线程');
    }
}
```


构造 Thread 类的时候会将 RunnableThread 实例传给 Thread 中，在调用 run 方法的时候，target 就不为 null，执行 RunnableThread 实例中的 run 方法。


### 1.1.3 线程池创建线程


```java
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```


`java.util.concurrent.Executors` 类中可以看到线程池的默认实现方式还是 new Thread() 实现。


### 1.1.4 有返回值的 Callable 创建线程


```
class CallableTask implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        return new Random().nextInt();
    }
}

//创建线程池
ExecutorService service = Executors.newFixedThreadPool(10);
//提交任务，并用 Future提交返回结果
Future<Integer> future = service.submit(new CallableTask());
```


submit 最终还是将任务放在了线程池中，而线程的创建仍然是基于构造 Thread 类实现的。
## 1.2 线程的中断
Java 中的线程中断是线程间的协作模式，通过设置线程的中断标志并不能直接终止该线程的执行 而是被中断的线程根据中断状态自行处理。
### interrupt
interrupt 中断线程，A 运行时，B 可以调用 A 的 interrupt 方法设置 A 的中断标志并立即返回。 **A 并没有中断，它会继续执行下去** 。


如果 A 调用了wait/sleep/join 等方法然后 B 调用了 interrupt，A 会抛出 `InterruptedException`  返回。当线程为了等待某些条件阻塞当前的线程，而在阻塞过程中条件被满足了就可以调用 interrupt 方法抛出异常而返回恢复到激活状态



| isInterrupted  | interrputed  |
| --- | --- |
| 不清除标记 | 清除标记 |
| thread.isinterrupted | Thread.interrupted（ **当前线程** ） |

```java
threadOne.interrupt();
System.out.println(threadOne.isInterrupted());	
// true

System.out.println(threadOne.interrputed());	
// false 虽然 threadOne 调用，但还是获取的当前线程也就是 main

System.out.println(Thread.interrputed());		
// false 与 threadOne.interrputed() 保持一致

System.out.println(threadOne.isInterrupted());	
// true threadOne 标记并灭有被清除
```
## 1.3 线程的生命周期
线程的生命周期一共 6 种状态：1.New，2.Runnable，3.Blocked，4.Waiting，5.Timed Waiting，6.Terminated
![img](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590117917491-e7c5a4fd-18c5-48ba-bcd5-da12e2896a51.png#align=left&display=inline&height=291&margin=%5Bobject%20Object%5D&name=image.png&originHeight=581&originWidth=856&size=79706&status=done&style=none&width=428)

### 1.3.1 New


表示线程被创建但尚未启动的状态：当我们用 new Thread() 新建一个线程时，如果线程没有开始运行 start() 方法，所以也没有开始执行 run() 方法里面的代码，那么此时它的状态就是 New。而一旦线程调用了 start()，它的状态就会从 New 变成 Runnable。


### 1.3.2 Runnable


对应操作系统线程状态中的两种状态，分别是 Running 和 Ready，也就是说，Java 中处于 Runnable 状态的线程有可能正在执行，也有可能没有正在执行，正在等待被分配 CPU 资源。


### 1.3.3 阻塞状态


#### Blocked
| 进入 Blocked | 进入 Runnable |
| --- | --- |
| 进入 synchronized 保护的代码没有抢到 **monitor 锁** | 抢到了 monitor锁 |



#### Waiting
| 进入 Waiting | 进入 Runnable | 进入 Blocked |
| --- | --- | --- |
| Object.wait() |  | Object.notify/notifyAll |
| Thread.join() | join 线程执行完毕/interrupt |  |
| LockSupport.park() | LockSupport.unpark() |  |



**Blocked 仅仅针对于 monitor 锁**，而 Java 中还有很多其他锁，比如 ReentrantLock 如果线程没有抢到该锁就会进入 waiting 状态，因为本质上是调用的 `LockSupport.park()` 所以看成情况3。


notify/notifyAll 会进入 Blocked 是因为调用方法前必须首先持有该 monitor 锁，所以处于 Waiting 状态的线程被唤醒时拿不到该锁，就会进入 Blocked 状态。


#### Timed Waiting
| 进入 Timed Waiting | 进入 Runnable | 进入 Blocked |
| --- | --- | --- |
| Object.wait(long timeout) |  | Object.notify/notifyAll |
| Thread.sleep(long millis) | 时间到/interrupt |  |
| Thread.join(long millis) | 时间到/join 线程执行完毕/interrupt |  |
| LockSupport.parkNanos(long nanos) | LockSupport.unpark() |  |
| LockSupport.parkUntil(long deadline) | LockSupport.unpark() |  |



### 1.3.4 Terminated


run() 方法结束或者出现未捕获的异常


## 1.4 wait/notify/notifyAll


- Object.wait()：释放当前对象锁，并进入阻塞队列
- Object.notify()：唤醒当前对象阻塞队列里的任一线程（并不保证唤醒哪一个）
- Object.notifyAll()：唤醒当前对象阻塞队列里的所有线程



每个对象里都有一个 monitor ，而 monitor 里面有一个该对象的锁和一个等待队列和一个同步队列


### 1.4.1 wait


在使用 wait 方法时，必须把 wait 方法写在 synchronized 保护的 while 代码块中，并始终判断执行条件是否满足。


> “wait method should always be used in a loop:
> synchronized (obj) {
>      while (condition does not hold)
>          obj.wait();
>      ... // Perform action appropriate to condition
> }
> This method should only be called by a thread that is the owner of this object's monitor.”



使用 synchronized 保护的根本原因就在于将 while 和 wait 组成原子操作，避免在 isEmpty 和 wait 方法之间执行了 notify 方法


```java
public class MyBlockingQueue {
    Queue<String> buffer = new LinkedList<String> ();
    public void give(String data) {
        synchronized(this) {
            buffer.add(data);
            notify();
        }
    }
    public String take() throws InterruptedException {
        synchronized(this) {
            // 使用 while 循环避免出现虚假唤醒的问题
            while (buffer.isEmpty()) {
                wait();
            }
            return buffer.remove();
        }
    }
}
```


### 1.4.2 wait 和 sleep


wait/notify/notifyAll 方法被定义在 Object 类，而 sleep 方法定义在 Thread 中的原因：


1. 每个对象都有 monitor 锁，并且由于所有的对象都可以上锁，所以作为所有对象父类的 Object 包含 wait/notify/notifyAll 就更加合理
1. 一个线程可以持有多把锁，如果让 Thread 去管理会带来很大的局限性
| wait | sleep |
| --- | --- |
| 阻塞线程 | 阻塞线程 |
| 可以响应 interrupt 中断 | 可以响应 interrupt 中断 |
| 需要获取 monitor 锁 | 无需 |
| 释放 monitor 锁 | 不释放锁 |
| **只能被中断和唤醒恢复** | 时间到期自动恢复 |
| Object | Thread |



wait/notify/notifyAll  方法调用前没有获取到 monitor 锁会抛出异常 IllegalMonitorStateException


**虚假唤醒**：一个线程可以从挂起状态变为可以运行状态（ 就是被唤醒），即使该线程没有被其他线程调用 notify/notifyAll方法进行通知，或者被中断，或者等待超时。


虚假唤醒就是调用 wait 方法是必须放在循环中重复判断的根本原因。


notify 需要加锁的原因：当获取到该对象的锁之后，才能去该对象对应 monitor 的等待队列去唤醒一个线程。值得注意的是，只有当执行唤醒工作的线程离开同步块，即释放锁之后，被唤醒线程才能去竞争锁。


## 1.5 join/sleep/yield


### 1.5.1 join
join 方法是 Thread 直接提供的，等待线程终止的方法。主线程调用 join 方法后被阻塞，等待线程执行完毕后返回。


```java
public void testJoin() {
    Thread main = Thread.currentThread();
    Thread threadA = new Thread(() -> {
        System.out.println("thread a is running!");
        // 死循环
        while (true) {

        }
    });

    Thread threadB = new Thread(() -> {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("thread b is running！");
        main.interrupt();
    });

    threadA.start();
    threadB.start();
    try {
        threadA.join();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("main is end");
}
```


主线程调用了 threadA.join **阻塞了自己**，等待线程 A 执行完毕，threadB 在等待 1s 后会调用主线程的 interrupt 方法设置主线程的中断标志，所以异常是出现在 join 方法处。


## 1.5.2 sleep


sleep 方法也是由 Thread 直接提供。调用线程会让出指定时间的执行权，在这期间不参与 CPU 的调度，但是**不会释放 monitor 锁**，时间到了正常返回，线程处于就绪状态。


睡眠期间如果被 interrupt 中断，会在 sleep 方法处抛出 InterruptException。


### 1.5.3 yield


yield 方法由 Thread 提供，暗示线程调度器让出自己的 CPU 的使用，**线程调度器可以忽略这个暗示**。当让出 CPU 使用权之后并不是状态是处于就绪 Ready。实际开发中这个方法很少使用，一般用于测试复现并发问题。


用 sleep 方法时调用线程会被阻塞挂起指定的时间，在这期间线程调度器不会去调度该线程，yield 方法线程只是让出自己剩余的时间片，并没有被阻塞挂起，而是处于就绪状态，线程调度器下一次调度时就有可能调度到当前线程执行。
## 1.6. 实现生产者消费者


### 1.6.1. wait/notify


```java
public class MyBlockQueue {
    private int max_size;
    private LinkedList storage;

    public MyBlockQueue(int max_size) {
        this.max_size = max_size;
        storage = new LinkedList();
    }

    public synchronized void put() throws InterruptedException {
        while (storage.size() == max_size) {
            System.out.println("max_size");
            wait();
        }
        storage.add(new Object());
        System.out.println("add Object now size:" + storage.size());
        notifyAll();
    }

    public synchronized void take() throws InterruptedException {
        while (storage.size() == 0) {
            System.out.println("0");
            wait();
        }
        storage.remove();
        System.out.println("remove Object now size:" + storage.size());
        notifyAll();
    }

    public int size(){
        return storage.size();
    }
}
@Test
public void testMyBlockedQueue() throws InterruptedException {
    MyBlockQueue myBlockQueue = new MyBlockQueue(10);
    Runnable producer = () -> {
        while (true) {
            try {
                myBlockQueue.put();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };

    Runnable consumer = () -> {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        while (true) {
            try {
                myBlockQueue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };

    new Thread(producer).start();
    new Thread(consumer).start();
    Thread thread = Thread.currentThread();
    thread.sleep(10000);
}
```


核心部分 MyBlockedQueue 中的 put 和 take 方法，队列的空和满为条件唤醒生产者和消费者


### 1.6.2. Condition（TODO）
# 2 什么是线程安全
多个线程 **同时读写** 一个共享资源并且没有任何同步措施导致脏数据或者其他不可预见的结果。
## 2.1 三种安全问题

1. 运行结果错误：线程拥有的时间片执行完成，导致线程安全问题
1. 发布或初始化：初始化所在线程在调用线程之后执行，导致调用对象为 null
1. 活跃性问题：
   1. 死锁：两个线程互相等待对方资源互不相让
   1. 活锁：线程并未发生阻塞，但一直获取不到结果所以始终需要执行
   1. 饥饿：某些资源始终获取不到，导致线程长时间得不到执行
## 2.2 多线程的性能开销

1. 上下文切换
1. 高速缓存失效
1. 协作开销：CPU 编译器重排序优化无法执行
## 2.3 线程切换
多线程数是大于 CPU 个数的，为了保持多线程是同步执行的，所以采取 **时间片轮转** 的策略。线程在时间片内占用 CPU 执行任务，时间片执行完，就会处于 就绪状态（Ready，Runnable 包含 Ready 和 Running 两种状态），并让 CPU 被其他线程占用这就是上下文切换。
## 2.5 线程死锁
两个以上的线程因资源争夺造成相互等待的现象被称为线程死锁。
一般来说想要避免死锁只需要破坏掉线程死锁的一个必要条件即可。保证资源的有序性就是解决线程死锁的方法。
## 2.6 守护线程与用户线程
守护线程和用户线程的最大区别在于守护线程在不会影响 JVM 的退出，而 JVM 退出的时候必须等到所有用户线程结束。 `setDaemon(true)` 可以设置守护线程


# 3 线程池
## 3.1 线程池基本概念
### 3.1.1 优点

1. 线程池管理线程的生命周期，降低了线程创建的消耗，并且始终有线程处于工作状态提高了响应速度
1. 线程池中协调内存和 CPU 资源，避免出现内存溢出或者 CPU 资源浪费的情况
1. 统一管理资源，便于统计
### 3.1.2 线程池结构

1. 线程池管理器：线程池的创建、销毁，添加任务
1. 工作线程：执行任务
1. 阻塞队列：线程池的缓冲机制
1. 任务：实现统一接口
### 3.1.3 线程池执行逻辑
![线程执行逻辑](https://gitee.com/mochi/image/raw/master/img/%E7%BA%BF%E7%A8%8B%E6%89%A7%E8%A1%8C%E9%80%BB%E8%BE%91.png)

## 3.2 六种线程池
| 参数 | FixedThreadPool | CachedThreadPool | ScheduledThreadPool | SingleThreadExecutor | SingleThreadScheduledExecutor |
| --- | --- | --- | --- | --- | --- |
| corePoolSize | 构造函数传入 | 1 | 构造函数传入 | 1 | 1 |
| maxPoolSize | 同 corePoolSize | Integer.MAX_VALUE | Integer.MAX_VALUE | 1 | Integer.MAX_VALUE |
| keepAliveTime | 0 | 60s | 0 | 0 | 0 |
| 队列 | LinkedBlockingQueue | SynchronousQueue | DelayedWorkQueue | LinkedBlockingQueue | DelayedWorkQueue |
| 特点 | 线程数量固定 | 线程无限 | 定时或周期 | 提交顺序执行 | Scheduled 的特例 |

### ScheduledThreadPool 的延时设置
```java
ScheduledExecutorService service = Executors.newScheduledThreadPool(10);
 
service.schedule(new Task(), 10, TimeUnit.SECONDS);
// 以开始时间为节点 
service.scheduleAtFixedRate(new Task(), 10, 10, TimeUnit.SECONDS);
// 以结束时间为节点
service.scheduleWithFixedDelay(new Task(), 10, 10, TimeUnit.SECONDS);
```
### ForkJoinPool
JDK 8 中加入的线程池，fork 分裂任务并执行，join 汇总结果，适合用于递归场景，比如树的遍历、最优路径搜索。


ForkJoinPool 除了一个共用队列外，每个子线程都有自己独立的 deque 队列（两端队列），降低了线程的竞争与切换。work-stealing 也可以利用 deque 平均线程的负载。「偷取」的线程先进先出，被「偷取」的线程则是后进先出。
## 3.3 四种拒绝策略
### 3.3.1 拒绝时间

1. 调用 shutdown 后，线程池内部没有执行完的任务正在执行，此时仍然有任务提交就会被拒绝
1. 线程池工作饱和，无法接受新的任务
### 3.3.2 拒绝策略

- AbortPolicy：抛出异常
- DiscardPolicy：丢弃任务
- DiscardOldestPolicy：丢弃存活时间最长的任务
- CallerRunsPolicy：将任务交给**提交任务的线程**执行「不是主线程」



CallerRunsPolicy 的优点：

1. 任务不会被丢弃
1. 交给提交任务的线程减缓任务提交速度，队列中任务又被消耗了。
## 3.4 阻塞队列

- LinkedBlockingQueue：容量无限
- SynchronousQueue：容量为 0 
- DelayedWorkQueue：按延迟时间长短排序，堆
- ArrayBlockingQueue：容量有限

ForkJoinPool 是使用的内部类 WorkQueue，最大为 1<<26 //64M
## 3.5 定制线程
### 3.5.1 线程数定制
#### CPU 密集型
加密、解密、压缩和计算等大量耗费 CPU 资源的任务建议 CPU 核心数的 1~2 倍    
#### IO 耗时型
数据库、文件读写和网络通信等不耗费 CPU 性能的任务
`线程数 = CPU 核心数 *（1+平均等待时间/平均工作时间`
### 3.5.2 阻塞队列
CPU 密集型，队列容量大，降低吞吐量
IO 耗时型，队列容量小
### 3.5.3 线程工厂
通过 Guava 的 ThreadFactoryBuilder 类创建线程工厂
```java
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
    // 核心线程数量
    6,
    // 最大线程数
    12,
    // 空闲超时一小时（调用频繁）
    3600, TimeUnit.SECONDS,
    // 阻塞队列
    new ArrayBlockingQueue<>(20100),
    // 线程工厂
    new ThreadFactoryBuilder().setNameFormat("my-pool-%d").build(),
    // 过多任务直接主线程处理
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```
### 3.5.4 拒绝策略
自定义拒绝策略需要实现 RejectedExecutionHandler 接口
```java
private static class CustomRejectionHandler implements RejectedExecutionHandler { 
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) { 
        //打印日志、暂存任务、重新执行等拒绝策略
    } 
}
```
## 3.6 关闭线程

- `void shutdown;` 执行完正在执行的任务和队列中等待的任务后才彻底关闭，此时提交方法均会被拒绝
- `boolean isShutdown;` 是否已经开始关闭线程池，true 不表示线程池被关闭
- `boolean isTerminated;` 线程池已关闭并且线程池内部是空的，所有剩余的任务都执行完毕
- `boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;`
- `List<Runnable> shutdownNow;` 给所有线程池中的线程发送 interrupt 中断信号，尝试中断这些任务的执行，然后会将任务队列中正在等待的所有任务转移到一个 List 中并返回



注意：线程应具有响应中断信号的能力，否则即便是调用了 shutdownNow 方法，线程依旧可能对于中断信号不理不睬。
## 3.7 线程复用原理
线程池将线程和任务解耦，使线程拥有了不断执行任务的能力。代码中使用 Work 类封装了 Thread 类，执行任务时也不是执行的 start 方法，而是直接去调用 Work 类中的 run 方法，从而是实现复用线程。
```java
runWorker(Worker w) {
    Runnable task = w.firstTask;
    while (task != null || (task = getTask()) != null) {
        try {
            task.run();
        } finally {
            task = null;
        }
    }
}
```
# 4 锁
## 4.1 偏向锁/轻量锁/重量锁
这三种锁用来描述 sychronized 锁的状态，通过对象头的 mark word 来表明锁的状态。

1. 偏向锁：资源不存在竞争，使用只需要打上标记即可，当线程获取锁的时候会记录下线程，如果获取锁的线程就是标记的线程，就直接获取锁。
1. 轻量锁：当线程出现和其他线程的竞争时，偏向锁就会通过 CAS 操作和自旋操作升级为轻量锁
1. 重量锁：利用操作系统机制实现的锁，用于多个线程实际竞争，并且获取不到锁的线程都会被阻塞
## 4.2 可重入锁/非可重入锁
可重入就是当前线程在获取了锁的情况下再次申请锁是否能够获取到，可重入锁中最典型的例子就是 ReentrantLock 
## 4.3 共享锁/独占锁
一把锁可以被多个线程共同拥有，一把锁只能被一个线程拥有，典型的例子就是读写锁。总的来说就是 **读读共享，其他互斥**  
### 4.3.1 读写锁的插队策略
```java
ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock(false);
ReentrantReadWriteLock.ReadLock readLock = readWriteLock.readLock();
ReentrantReadWriteLock.WriteLock writeLock = readWriteLock.writeLock();
```
读写锁的讨论还得是建立在公平锁的基础上，默认锁是 false （非公平）。在非公平锁的情况下，到下个节点是**写锁那么就不允许插队，但是如果是读锁就可以插队。**这么做只要是为了防止大量的读操作不停插队到写线程线程饥饿。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1471883/1592637021846-16c72e80-b926-47ad-9187-60a96c202767.png#align=left&display=inline&height=354&margin=%5Bobject%20Object%5D&name=image.png&originHeight=708&originWidth=1742&size=150662&status=done&style=none&width=871)
公平锁就简单的多，所有线程按照请求顺序依次执行，此时读线程共享的特性不会改变。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1471883/1592637082700-b934dcb9-2198-4da2-ac42-d69ab8ca3170.png#align=left&display=inline&height=245&margin=%5Bobject%20Object%5D&name=image.png&originHeight=490&originWidth=1470&size=89444&status=done&style=none&width=735)
### 4.3.2 锁的降级
写锁可以降级为读锁，但是读锁不可以升级为写锁，因为读锁是共享的可能存在多个读线程同时升级为写锁导致死锁的状况。（读锁的升级可以实现，需要保证升级的时候只有一个线程，但是 ReentrantReadWriteLock 无法实现）
降级的原因：一直持有写锁确实可以保证线程安全，但是没有必要，因为持有写锁也就意味着没有办法实现多个线程同时读取，所以锁降级是一个很好的方案来解决性能问题。

## 4.4 乐观锁和悲观锁
悲观锁要求在获取资源之前必须先获取锁，而乐观锁则不需要，乐观锁避免数据问题时通过 CAS 实现


### 4.4.1 悲观锁
案例：synchronized 和 Lock，适用于并发写入多，竞争激烈的场景，可以避免大量无用的反复尝试
#### synchronized 和 Lock
##### 相同点

- 可重入
- 保证线程安全
- 内存可见性
##### 不同点

1. synchronized 加解锁都是隐式的，而 lock 必须显示的加解锁
1. synchronized 时不可中断锁，而 lock 可以中断
1. synchronized 解锁的顺序必须一致，而 lock 加解锁更加灵活
1. synchronized 只能被一个线程拥有，而 lock 实现类 读锁就可以多个线程共享一个锁
1. synchronized 在JVM 优化后性能不比 lock 差
### 4.4.2 乐观锁
案例：Atomic，适用于读取多的场景，写入多也适合，不加锁的特性可以大幅提高性能，但是前提是并发不能激烈否则会造成大量的无用尝试
## 4.5 公平锁和非公平锁
公平锁和非公平锁的区别在于在运行任务时，公平锁严格按照请求顺序执行，注意：并不时说非公平锁就时完全的先到先得，只是会存在一定几率的不按照顺序执行。当前一个线程释放锁的瞬间，恰好有线程请求锁，那么这时候就可以直接获取锁。
在 ReentrantLock 中非公平锁就是默认的策略，核心原因就是在于当前 A 线程释放锁的瞬间，有线程 C 请求，直接将 CPU 分配给请求 C 线程就节约了唤醒等待线程 B 的这个时间，这样就提高了运行效率。
```java
 Lock lock = new ReentrantLock(true);	// 公平锁
```
|  | 优点 | 缺点 |
| --- | --- | --- |
| 公平锁 | 线程按序执行 | 速度慢、吞吐小 |
| 非公平锁 | 更快、吞吐大 | 容易造成线程饥饿 |

## 4.6 自旋锁和非自旋锁
自旋锁指的时线程在获取不到锁的时候不会陷入阻塞或者释放 CPU 资源，而是不断自旋（循环获取锁），自旋锁
`AtomicLong#getAndAdd()` 就是应用了自旋的逻辑，AtomicLong 也正是通过 CAS 和 自旋实现的线程安全。

```java
public final long getAndAddLong(Object var1, long var2, long var4) {
    long var6;
    do {
        var6 = this.getLongVolatile(var1, var2);
    } while(!this.compareAndSwapLong(var1, var2, var6, var6 + var4));

    return var6;
}
```
### 4.6.1 自旋的优缺点
自旋的主要目的是为了通过自旋的方式避免线程切换的开销，但是同样也有可能造成新的问题，如果一直没有办法获取到锁，那么 CPU 性能损耗就有可能超过线程切换的开销。

自旋锁适用于并发度不高，临界范围小的场景，**如果线程占用锁的时间长，那么不适合使用自旋锁。**

### 4.6.2 自定义一个 ReentrantLock 
通过 while 循环实现了自旋锁，再配合 CAS 实现了一个自定义的可重入锁，借助于 AotmicReference 类装载获取到锁的线程，以 count 变量计算重入次数。

```java
    /**
     * 重入次数 count
     */
    private int count = 0;
    /**
     * 原子引用类型：装载线程，判断是否属于锁的拥有者发起的请求。
     */
    private AtomicReference<Thread> atomicReference = new AtomicReference();

    /**
     * 加锁
     */
    public void lock() {
        Thread thread = Thread.currentThread();
        // 当前线程重入操作
        if (thread == atomicReference.get()) {
            count ++;
            System.out.printf("%s 增加重入次数：%s \n", Thread.currentThread().getName(), count);
            return;
        }
        // 自旋获取锁
        while (!atomicReference.compareAndSet(null, thread)) {
            System.out.printf("%s 线程自旋获取锁中... \n", Thread.currentThread().getName());
        }
    }

    public void unlock() {
        Thread thread = Thread.currentThread();
        if (thread == atomicReference.get()) {
            if (count > 0) {
                System.out.printf("%s 减少重入锁次数：%s\n", Thread.currentThread().getName(), count);
                count--;
            } else {
                atomicReference.set(null);
                System.out.printf("%s 释放锁\n", Thread.currentThread().getName());
            }
        }
    }
```

## 4.7 可中断锁和不可中断锁
synchronized 关键字修饰的锁代表不可中断锁，一旦申请了锁就必须等到获取到锁才能执行，而可中断锁（ReentrantLock ）就可以在中断时去执行别的事情。


## 4.8 synchronized 和 volatile
### 4.8.1 synchronized 
synchronized 是一种原子内置锁，Java 中每个对象都可以把它当做同步锁使用，事实上每个对象内部都有一个 monitor 锁，在进入 synchronized 代码块前会获取锁，正常退出，抛出异常或者 wait 方法都会释放锁。monitor 锁是排它锁，一旦获取其他线程必须等待。


tips: 静态的 synchronized 代码块以 class 对象为锁


synchronized 耗时的主要原因有2点：

1. 需要从用户态切换到内核态执行阻塞
1. 导致上下文切换



> [用户态和内核态](https://www.cnblogs.com/maxigang/p/9041080.html)



进入 synchronized 块就是把 synchronized 块内使用的变量从线程的工作空间中清除，这样获取变量就不会从工作空间获取，而是直接从主内存获取。退出 synchronized 块就是将变量的改变刷新到主内存。


synchronized 除了可以处理共享变量内存可见性问题，还可以实现原子性操作。


### 4.8.2 volatile
volatile 是相对于 synchronized 更加轻量的解决共享变量内存可见性的解决方案。使用 volatile 修饰的变量在写入时不会把值缓存在工作空间而是直接刷新到主内存，读取的时候也是从主内存中获取（先清空工作空间缓存）。
**此外 volatile 还有一种用法就是：禁止指令重排序优化。**一般我们写的代码都会被编译器优化后再生成 class 文件，而编译器并不能保证我们的代码顺序和之前一致，只能保证结果一致。


注意：volatile 禁止指令重排序优化是在 1.5 版本以后才生效


volatile 其实和 synchronized 是一个实现逻辑，写入就相当于退出 synchronized 代码块，读取就相当于进入 synchronized 代码块。


#### 不使用 volatile 的情况

1. 依赖于变量当前值，因为 volatile 不能保证原子性，所以多部操作的时候不能使用
1. 已经被 synchronized 修饰，本身已经保障了可见性无需再使用 volatile



#### java 指令重排序
java 内存模型允许编译器和处理器对指令重排序以提高运行性能，并且只会对不存在数据依赖性的指令重排序
#### 伪共享
通过 **字节填充** 的方式避免伪共享的问题，创建变量时使用字符填充变量所在的缓存行，从而避免多个变量存放在一个缓存行。 
### 4.8.3 综合比较
| synchronized | volatile |
| --- | --- |
| 原子性操作 | 不保证原子性 |
| 阻塞，切换上下文 | 非阻塞算法 |

## 4.9 lock
lock 锁是 JAVA 5 中引入的接口，最常用的是 ReentrantLock，一般我们都会把 lock 和 synchronized 锁进行比较，事实上 lock 锁并不是为了替换 synchronized 而存在的，lock 锁可以提供更多种的功能。一般情况下 lock 锁只允许一个线程来访问共享资，当然也有例外比如读锁就可以同时持有。
### 4.9.1 lock
lock() 方法就是获取锁的方法，要注意 **lock 方法获取的锁必须显示的释放**，而synchronized 锁则是 JVM 实现的释放和加锁，所以一般情况下没有特殊的需求，使用 synchronized 锁无论是从性能还是代码安全角度都是更合适的。 **lock 方法不能中断** ，陷入死锁后就会陷入永久等待。
```java
lock.lock();
try{
    // 获取到了锁的资源
}finally{
	// 释放锁
    lock.unlock();   
}
```
### 4.9.2 tryLock
tryLock() 相对于 lock() 最大的改变就是当获取不到锁的时候会立刻返回一个 false ，不至于一直等待。相对于 lock() 方法而言，tryLock() 就不会陷入等待，也因此不会陷入死锁问题，因为一旦获取不到锁对象还是会继续执行未获取到锁的方法。
```java
if(lock.tryLock()){
    try{
        // 获取到了锁的资源
    }finally{
        // 释放锁
        lock.unlock();   
    }
} else {
// 未获取到锁的业务逻辑
}
```
此外 tryLock() 方法还可以传递一个 waitingTime 参数，在这段时间结束后再去获取锁返回 boolean 值，注意: **tryLock(time, TimeUnit) 在等待期间是可以随时中断的**  
```java
if (lockB.tryLock()) {
    try {
        System.out.println("B get BLock");
        if (lockA.tryLock(2000 ,TimeUnit.MILLISECONDS)) {
            try {
                System.out.println("B get ALock");
            } finally {
                System.out.println("B release ALock");
                lockA.unlock();
            }
        } else {
            System.out.println("B don't get ALock");
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        System.out.println("B release BLock");
        lockB.unlock();
    }
}else {
    System.out.println("B don't get BLock");
}
```
### 4.9.3 lockInterruptibly
lockInterruptibly() 可以理解为是 lock() 的可中断版本，在加锁的同时是可以响应中断的。
```java
try {
    lock.lockInterruptibly();
    try {
        System.out.println("A get BLock");
    } finally {
        System.out.println("A release BLock");
        lock.unlock();
    }
} catch (InterruptedException e) {
    System.out.println("A response Interrupt");
    e.printStackTrace();
}
```
### 4.9.4 unlock
unlock() 就是解锁，当然对于可重入锁 ReentrantLock 而言，unlock() 就是将内部锁持有器做 -1 操作，当计数器为 0 锁就被真正的释放了。

## 4.10 JVM 对于锁的优化
### 4.10.1 锁消除
如果发现对象不可能被多个线程访问，那么 JVM 就会将其当做栈上数据（栈中的数据只能被当前线程访问）同时消除锁。

核心理念在于减少不必要的加锁和解锁操作，提高整体的效率
### 4.10.2 锁粗化
代码块中释放锁后没有执行任何操作又执行了获取锁操作，JVM 就会针对这种情况扩大同步区，但是这样也并非没有问题，比如循环中就不适合使用锁粗化。

锁粗化功能是默认打开的，用 `-XX:-EliminateLocks` 可以关闭该功能。

### 4.10.3 自旋锁的自适应
为了避免自旋锁自旋过长，JDK 1.6 引入了自适应的自旋锁来解决这个问题，自旋的持续时间是变化的，会根据自旋成功率失败率等修改持续时间。
# 5 并发容器
## 5.1 ConcurrentHashMap
ConcurrentHashMap 可以理解为**线程安全、并发效率高**的 HashMap。
### 5.1.1 ConcurrentHashMap 在 1.7 和 1.8 中的比较
![ebae1767a0811bf36814774dce485ee1.png](evernotecid://1B6BE08F-E57B-42A6-995F-1020670DD7F8/appyinxiangcom/10082257/ENResource/p399)

ConcurrentHashMap 1.7 时底层采用的是 segment 分段锁的结构，加锁也是针对于 segment 分段加锁，默认 segment 大小是 16 所以最多支持 16 个线程并发操作，并且这个值在初始化后是不可以扩展的。segment 锁的实现也是继承的 ReetrantLock 锁，对于 Hash 碰撞采用的是拉链法。

![a43cafd16ec6f85c22ce4ba381c9ac77.png](evernotecid://1B6BE08F-E57B-42A6-995F-1020670DD7F8/appyinxiangcom/10082257/ENResource/p400)

1.8 版本底层结构改成了数组、链表加红黑树的形式与 HashMap 类似。加锁则是针对于节点加锁，锁的粒度更细了，所以支持的最大线程操作数就是数组的大小，对于 Hash 碰撞首先采用拉链法，在长度超过阈值后就会转换为红黑树。

### 5.1.2 ConcurrentHashMap 和 HashTable
ConcurrentHashMap 相对于 HashTable 性能更高，锁的粒度更小，锁只针对于节点，而 HashTable 就是 HashMap 的相关方法上加上 synchronized 关键字保证线程安全，锁住的是整个对象。

而 ConcurrentHashMap 则是 CAS + synchronized + Node 实现的线程安全。

**ConcurrentHashMap 支持迭代时修改，而 HashTable 不允许修改**
### 5.1.2 ConcurrentHashMap 和 HashMap
HashMap 和 ConcurrentHashMap 基本结构相同
HashMap 支持 key 和 value 为 null，而 ConcurrentHashMap 均不允许为 null，如果有 null 就会抛出 NullPointerException。

#### HashMap 常见的线程不安全的几个原因
1. 扩容期间取值错误
2. 同时 put 操作导致数据丢失，数据 bucket 位置一致，导致一个数据丢失
3. 可见性无法保证，线程 A 的操作对于线程 B 不可见

#### 阈值 8 
无论是 HashMap 还是 ConcurrentHashMap 在链表长度到 8 之后都会由链表转换为 红黑树。
**通常 Hash 算法正常的情况下，链表的长度不会很长，红黑树反而会增加空间负担，当节点长度要到 8 的概率小于 千万分之一**，所以将 8 设为阈值。

红黑树的设计更多的是为了解决开发人员重写的 Hash 算法不好，导致 HashMap 的查询效率降低。
## 5.2 CopyOnWriteArrayList
采用读写分离的思想实现，读和写时使用不同的容器。
### 5.2.1 适合使用场景
1. 读多写少
2. 读操作要快，写操作可以慢点

实际开发中黑名单和系统级别的信息都适用于此，一旦加载就会被频繁访问，而写入次数相对少了很多。
### 5.2.2 容器特性
**读取完全不加锁，写入也不阻塞读操作**，只有写写互斥，但在写入的是允许读取。

当容器被修改的时候复制当前数组，所有的操作只针对于复制的数组，完成后将引用指向复制的数组。

迭代是允许修改集合

### 5.2.3 缺点
1. 内存占用高
2. 元素多开销大
3. 数据一致性问题

# 6 阻塞队列
## 6.1 什么是阻塞队列
阻塞队列就是 BlockingQueue，也就是线程安全的队列，一般用于解决业务中的线程安全问题，最典型的应用就是生产者和消费者，将复杂的线程安全问题交给队列管理，简化了开发难度。

同时队列还能实现具体任务和执行类之间的解耦，任务在阻塞队列中，而存放任务的线程就不需要依赖于具体执行类。

> Deque 的意思是双端队列，音标是 [dek]，是 double-ended-queue 的缩写，它从头和尾都能添加和删除元素；而普通的 Queue 只能从一端进入，另一端出去。这是 Deque 和 Queue 的不同之处，Deque 其他方面的性质都和 Queue 类似

## 6.2 常用方法
- 抛出异常：add、remove、element
- 返回结果但不抛出异常：offer、poll、peek
- 阻塞：put、take

| 方法      | 说明       | 特点                          |
|---------|----------|-----------------------------|
| add     | 添加元素     | 队列满了，illegalStateException  |
| remove  | 返回并删除头元素 | 队列为空，NoSuchElementException |
| element | 返回头元素    | 队列为空，NoSuchElementException |
| offer   | 添加元素     | 队列满了,false;队列不满，true        |
| poll    | 返回并删除头元素 | 队列为空，null                   |
| peek    | 返回头元素    | 队列为空，null                   |
| put     | 添加元素     | 队列满了，阻塞                     |
| take    | 返回并删除头元素 | 队列为空，阻塞                     |

#### put 为什么可以响应 interrupt 
因为 put 方法中使用的是 `lock.lockInterruptibly();` 来加锁的，所以可以响应中断

## 6.3 四种常见阻塞队列
- ArrayBlockingQueue：有界队列并且不可以扩容，基于数组存储元素，先进先出的顺序
- LinkedBlokingQueue：无界队列，底层数据结构基于内部用链表实现，先进先出的顺序
- DelayQueue：支持延迟的无界队列，元素会根据延迟时间的长短被放到队列的不同位置，越靠近队列头代表越早过期，借助 PriorityQueue 实现
- SynchronousQueue：**容量为 0**，无论取还是存数据都会先陷入阻塞
- PriorityBlockingQueue：支持优先级的无界阻塞队列
## 6.4 阻塞队列与非阻塞队列
Condition https://www.jianshu.com/p/28387056eeb4
阻塞队列底层都是使用 ReentrantLock 实现线程安全的，LinkedBlockingQueue 特殊点，采用了两把锁分别锁住头和尾。
### 6.4.1 ArrayBlockingQueue 的阻塞原理
ArrayBlockingQueue 实现并发同步的原理就是利用 ReentrantLock 和两个 Condition，读和写都需要先获取到 ReentrantLock 独占锁才能进行下一步操作。

进行读操作时如果队列为空，线程就会进入到读线程专属的 notEmpty 的 Condition 的队列中去排队，等待写线程写入新的元素；反之就是进入 notFull 中。
### 6.4.2 ConcurrentLinkedQueue 非阻塞线程安全的原理
ConcurrentLinkedQueue 借助无限循环和CAS方法实现线程安全。
## 6.5 选择阻塞队列的策略
功能、容量、能否扩容、内存结构和性能
1. 如果需要延迟功能就需要选择 DelayQueue，要是需要支持优先级的排序就得使用 PriorityBlockingQueue
2. 根据任务需求判定是否需要无限容量，是否需要扩容
3. 具体在业务中适合什么结构，性能要求有多高来确定

# 7 原子类
## 7.1 原子类简介
原子类顾名思义就是基本的操作都保证原子性的类，原子类和锁的作用类似，都是为了保证并发时的线程安全。
原子类相对于锁有一定优势：
1. 原子类粒度更细，可以把竞争缩小到变量级别，通常情况下锁的粒度都要更大
2. 原子类效率更高，原子类底层是利用了 CAS 不会阻塞线程。


| 类型	 | 具体类|
|---|---|
|Atomic* 基本类型原子类 | AtomicInteger、AtomicLong、AtomicBoolean |
| Atomic*Array 数组类型原子类	 | AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray |
| Atomic*Reference 引用类型原子类 | 	AtomicReference、AtomicStampedReference、AtomicMarkableReference |
| Atomic*FieldUpdater 升级类型原子类 | 	AtomicIntegerfieldupdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater |
| Adder 累加器 | 	LongAdder、DoubleAdder |
| Accumulator 积累器	 | LongAccumulator、DoubleAccumulator |
### 7.1.1 基本类型原子类
基本类型包括 AtomicInteger、AtomicLong 和 AtomicBoolean 三种，对应的方法分别有
- get() 获取当前值
- getAndIncrement 和 getAndDecrement 自增自减
- getAndAdd(int i) 获取当前值增加 i
- compareAndSet(int except，int i) 如果 except 和预期值相等使用 i 进行替换  
### 7.1.2 数组类型原子类

数组类型的原子类相当于将基本类型的原子类聚合起来组成一个数组

- AtomicIntegerArray
- AtomicLongArray
- AtomicReferenceArray

### 7.1.3 引用类型原子类

保证对象的原子性

- AtomicStampedReference ：在 AtomicReference 的基础上增加了时间戳，用于解决 ABA 问题
- AtomicMarkableRefernece：在 AtomicReference 基础上增加布尔值，表示该对象已经删除

### 7.1.4 升级类型

原子更新器分别对应 Integer 、Long 和 Reference 三种，主要用于将已经声明的对象升级使之具有 CAS 操作的能力，之所以提供此方法主要是两个方面考虑：

1. 历史原因，修改成本很高，可以使用升级的原子类
2. 大部分场景不需要原子性，极少数情况下才需要用到，节约了内存

```java
public class AtomicUpdaterDemo implements Runnable{
    /**
     * 静态成员变量才可以被不同线程访问
     */
    static Score math;
    static Score chinese;

  	// 初始化依赖于静态方法
    static AtomicIntegerFieldUpdater<Score> scoreAtomicIntegerFieldUpdater =
            AtomicIntegerFieldUpdater.newUpdater(Score.class, "score");

    @Test
    public void testAtomicUpdater() throws InterruptedException {
        math = new Score();
        chinese = new Score();
        AtomicUpdaterDemo atomicUpdaterDemo = new AtomicUpdaterDemo();
        Thread t1 = new Thread(atomicUpdaterDemo);
        Thread t2 = new Thread(atomicUpdaterDemo);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("普通变量的结果：" + chinese.score);
        System.out.println("升级后的结果：" + math.score);
    }

    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            math.score ++;
            scoreAtomicIntegerFieldUpdater.incrementAndGet(chinese);
        }
    }

    static class Score {
        /**
         * java.lang.ExceptionInInitializerError
         * Caused by: java.lang.IllegalArgumentException: Must be volatile type
         */
        volatile int score;
    }
}
```

## 7.2 原子类的性能分析
### 7.2.1 Unsafe

Unsafe 是 CAS 的核心类，由于 Java 无法直接访问底层操作系统，而是通过 native 方法去实现，在 JDK 中 Unsafe 类就提供了硬件级别的原子操作，可以通过 Unsafe 类直接操作内存数据。

### 7.2.2 以 AtomicInteger 为例分析 Java 如何利用 CAS 实现原子操作

```java
public class AtomicInteger extends Number implements java.io.Serializable {
   // setup to use Unsafe.compareAndSwapInt for updates
   private static final Unsafe unsafe = Unsafe.getUnsafe();
   private static final long valueOffset;
 
   static {
       try {
           valueOffset = unsafe.objectFieldOffset
               (AtomicInteger.class.getDeclaredField("value"));
       } catch (Exception ex) { throw new Error(ex); }
   }
 	// volatile 修饰保证所有线程读取的值相同
   private volatile int value;
   public final int get() {return value;}
   public final int getAndAdd(int delta) {
    	 return unsafe.getAndAddInt(this, valueOffset, delta);
	 }
}
```

当 AtomicInteger 类被加载的时候就会调用静态代码块中的内容给 valueOffset 赋值，从而得到了当前原子类的偏移量「偏移地址就是计算机里的[内存分段](https://baike.baidu.com/item/内存分段/13017874)后,在段内某一地址相对于段首地址([段地址](https://baike.baidu.com/item/段地址))的[偏移量](https://baike.baidu.com/item/偏移量)」Unsafe 类就可以通过内存偏移地址获取数据的原值。

```java
public final int getAndAddInt(Object object, long offset, int delta) {
   int expectedValue;
   do {
       expectedValue = this.getIntVolatile(object, offset);
   } while(!this.compareAndSwapInt(object, offset, expectedValue, expectedValue + delta));
   return expectedValue;
}
```

这边修改了一下方法参数名便于理解，首先根据偏移地址获取内存中的值，然后比较是否相等，相等就替换如果被其他线程修改就接着循环直到修改成功。

## 7.3 AtomicInteger 性能问题
AtomicInteger 在高并发场景下的性能并不好，原因在于 AtomicInteger 中被 volatile 修饰的 value 变量需要频繁的 flush 和 reflush 操作才能保证内存可见性，所以导致性能不佳。

LongAdder 就解决了这个问题，采用 Cell[] 数组和 base 参数优化性能。竞争不激烈的情况直接操作 base 进行加减，而在竞争激烈的情况下就利用 Cell[]，通过计算出线程的 Hash 将线程分配到不同的 Cell 上进行修改，这样线程之间本质就不存在竞争关系。这种就是典型的空间换时间的案例，类似于 ConcurrentHashMap Segment 的使用。

**Atomic 适用于不仅仅使用简单加和减还需要使用到 CAS 操作的时候；LongAdder 则是更多的用于统计求和计数的场景。**

## 7.4 原子类与 volatile 的区别

volatile 主要作用就是防止指令重排序，保证内存可见性
![volatile 原理](https://gitee.com/mochi/image/raw/master/img/volatile%20%E5%8E%9F%E7%90%86.png)

但是 volatile 并不能保证操作的原子性，所以累加操作更加适合使用原子类去执行。volatile 更多用于修饰布尔值，以为赋值操作具备原子性，再加上内存可见就可以保障线程安全。

原子类更多多用于计数器的场景，不仅是简单的赋值还有一定的修改。

## 7.5 原子类 和 Synchronized 的区别
原子类本质是乐观锁，通过 while 循环和 CAS 操作保障对象操作的原子性，使用场景局限性比较大，虽然不会造成阻塞，但时间长了消耗资源也很多，适合于竞争不激烈的场景。

synchronized 关键字则是典型的悲观锁，利用管程「Monitor」来保障线程安全，使用场景更加广泛，并且在竞争不激烈的情况下 synchronized 的性能也是得到了优化的，并非直接阻塞。

## 7.65 Accumulator
Accumulator 适用于 大量并行计算的场景，可以利用线程池来提高计算效率，并且计算过程中的顺序不影响最终结果



# 8 线程本地变量
## 8.1 ThreadLocal 
### 8.1.1 使用场景



线程本地变量，每个线程都会创建一个这变量的本地副本，从而避免线程安全问题。

### 实现原理
ThreadLocal 本质上是 一个马甲，其本质是调用传入线程的 ThreadLocalMap 来存储变量，所以可以做到每个线程单独存储自己的内容。 **Thread 在父类设置，子线程获取不到**
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590475109296-c1aa9a97-c34e-4095-ba70-be30b515e48d.png#align=left&display=inline&height=390&margin=%5Bobject%20Object%5D&name=image.png&originHeight=779&originWidth=1181&size=34392&status=done&style=none&width=590.5)
每个线程内部都有一个 HashMap 类型的 threadLocals 变量，key 为 ThreadLocal 变量的引用， value 为我们自己设置的值。之所以是 Map 类型是因为可能有多个 ThreadLocal 变量。




## inheritableThreadLocal
可以让子线程可以访问父线程中设置的本地变量


### 并发与并行
| 并发 | 并行 |
| --- | --- |
| 多个任务同一 **时间段** 执行 | 多个任务 **同时** 执行 |

# Java 内存模型
### Java 内存模型详解
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590479481956-72397efc-2ee5-4110-925d-d585fab7af1e.png#align=left&display=inline&height=225&margin=%5Bobject%20Object%5D&name=image.png&originHeight=450&originWidth=536&size=23363&status=done&style=none&width=268)
所有变量保存在主内存中，线程需要使用时，将变量复制到自己的工作空间，线程读写变量都是操作的自己工作空间中的变量，最后更新到主内存中。
Java 的内存模型是一个抽象概念，实际在运行中还是有差别的。


tips: CPU 去主内存获取变量然后将变量所在内存区域的一个 Cache行 大小的内存复制到工作空间中，所以可能会把多个变量放到工作空间中。缓存和内存交换数据的单位就是**缓存行。**程序运行的局部性原理也就促使了数组在访问的时候多个元素都会被放入同一个缓存行，所以执行速度更快。但是在多线程并发修改一个缓存行中的多个变量时就会竞争缓存行，降低运行性能。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590477277145-37781a36-3adb-4414-b4ac-bc9c6d1bb64b.png#align=left&display=inline&height=292&margin=%5Bobject%20Object%5D&name=image.png&originHeight=583&originWidth=659&size=23532&status=done&style=none&width=329.5)
线程安全问题就是由于 L1 中的缓存是和线程关联的，当 L2 的缓存被其他线程刷新了就会和其他 CPU 中 L1 缓存的值产生冲突。
**对于可能被多个线程同时访问的可变变量，访问时都必须要持有同一个锁（无论读写）。**
### 无状态对象
**无状态对象一定是线程安全的，**以 Servlet 为例，Servlet 是单例，绝大多数情况下都是无状态的，既不包含任何域，也不包含对其他类中域的引用。临时状态也是保存在线程栈中的局部变量，所以线程直接不会相互影响，也就不存在共享状态。
但是当 Servlet 需要保存信息时，线程安全就需要考虑了。
## 线程概念
### 原子性
```java
    private Long value;

    public synchronized Long getValue() {
        return value;
    }

    public synchronized void incValue() {
        ++value;
    }
```
`++value` 不是原子操作事实上包含了 3 个操作分别是：读取 value，加一和写入 value，所以需要使用 synchronized 。判断并发的原子性是针对于 CPU 指令，而不是高级语言的语句，实际开发过程张可以通过 `java -c` 查看汇编语言是否是原子操作。**原子性的根本目的就是为了保持状态的一致性，在单个原子操作中更新所有的变量。**


getValue() 既然是读操作，不是 incValue 那样需要保证原子性，也不存在现场安全问题，是否可以去除 synchronized 关键字？不可以，因为这边的 synchronized 是为了保证内存可见性。


`java.util.concurrent.atomic` 包下包含了一些原子类能够确保对于 value 的操作都是原子的，这边我们可以是用 AtomicLong 代替 Long，++ 操作可以使用 `incrementAndGet()` 方法替代。**原子类尽量不要和同步代码块一起使用。**
### CAS
CAS 是 JDK 提供的非阻塞原子性操作，通过**硬件**保证了 比较-更新操作的原子性。具体操作就是如果对象 obj 的内存偏移量值是 except，则用新值替换旧的值 except。


ABA 问题在于表量的状态值产生了环形变化，解决方法可以增加一个 版本号 ，每次更新都修改版本号。


### 加锁机制
#### 重入
当线程 A 请求线程 B 持有的锁，请求就会被阻塞，那线程 A 请求 A 持有的锁还需要被阻塞吗？答案是不需要，monitor 锁获取的操作的粒度是**线程**，而不是调用**。**


当线程 A 再次请求自己持有的锁会将计数器累加，同理在释放锁的时候也需要计数器清 0 才可以释放锁。


**重入的原因**：如果父类和子类的方法都用 Synchronized 去修饰，那么当子类调用父类的方法就可以顺利获取到锁。如过不支持重入，那么子类将永远无法获取到锁。




# 10 面试题汇总

1. 为何说只有 1 种实现线程的方法？
1. 如何正确停止线程？为什么 volatile 标记位的停止方法是错误的？
1. 线程是如何在 6 种状态之间转换的？
1. wait/notify/notifyAll 方法的使用注意事项？
1. 有哪几种实现生产者消费者模式的方法？
1. 一共有哪 3 类线程安全问题？
1. 哪些场景需要额外注意线程安全问题？
1. 为什么多线程会带来性能问题？
1. 使用线程池比手动创建线程好在哪里？
1. 线程池的各个参数的含义？
1. 线程池有哪 4 种拒绝策略？
1. 有哪 6 种常见的线程池？什么是 Java8 的 ForkJoinPool？
1. 线程池常用的阻塞队列有哪些？
1. 为什么不应该自动创建线程池？
1. 合适的线程数量是多少？CPU 核心数和线程数的关系？
1. 如何根据实际需要，定制自己的线程池？
1. 如何正确关闭线程池？shutdown 和 shutdownNow 的区别？
1. 线程池实现“线程复用”的原理？
1. 你知道哪几种锁？分别有什么特点？
1. 悲观锁和乐观锁的本质是什么？
1. 如何看到 synchronized 背后的“monitor 锁”？
1. synchronized 和 Lock 孰优孰劣，如何选择？
1. Lock 有哪几个常用方法？分别有什么用？
1. 讲一讲公平锁和非公平锁，为什么要“非公平”？
1. 读写锁 ReadWriteLock 获取锁有哪些规则？
1. 读锁应该插队吗？什么是读写锁的升降级？
1. 什么是自旋锁？自旋的好处和后果是什么呢？
1. JVM 对锁进行了哪些优化？
1. HashMap 为什么是线程不安全的？
1. ConcurrentHashMap 在 Java7 和 8 有何不同？
1. 为什么 Map 桶中超过 8 个才转为红黑树？
1. 同样是线程安全，ConcurrentHashMap 和 Hashtable 的区别？
1. CopyOnWriteArrayList 有什么特点？
1. 什么是阻塞队列？
1. 阻塞队列包含哪些常用的方法？add、offer、put 等方法的区别？
1. 有哪几种常见的阻塞队列？
1. 阻塞和非阻塞队列的并发安全原理是什么？
1. 如何选择适合自己的阻塞队列？
1. 原子类是如何利用 CAS 保证线程安全的？
1. AtomicInteger 在高并发下性能不好，如何解决？为什么？
1. 原子类和 volatile 有什么异同？
1. AtomicInteger 和 synchronized 的异同点？
1. Java 8 中 Adder 和 Accumulator 有什么区别？
1. ThreadLocal 适合用在哪些实际生产的场景中？
1. ThreadLocal 是用来解决共享资源的多线程访问的问题吗？
1. 多个 ThreadLocal 在 Thread 中的 threadlocals 里是怎么存储的？
1. 内存泄漏——为何每次用完 ThreadLocal 都要调用 remove()？
1. Callable 和 Runnable 的不同？
1. Future 的主要功能是什么？
1. 使用 Future 有哪些注意点？Future 产生新的线程了吗？
1. 如何利用 CompletableFuture 实现“旅游平台”问题？
1. 信号量能被 FixedThreadPool 替代吗？
1. CountDownLatch 是如何安排线程执行顺序的？
1. CyclicBarrier 和 CountdownLatch 有什么异同？
1. Condition、object.wait() 和 notify() 的关系？
1. 讲一讲什么是 Java 内存模型？
1. 什么是指令重排序？为什么要重排序？
1. Java 中的原子操作有哪些注意事项？
1. 什么是“内存可见性”问题？
1. 主内存和工作内存的关系？
1. 什么是 happens-before 规则？
1. volatile 的作用是什么？与 synchronized 有什么异同？
1. 单例模式的双重检查锁模式为什么必须加 volatile？
1. 你知道什么是 CAS 吗？
1. CAS 和乐观锁的关系，什么时候会用到 CAS？
1. CAS 有什么缺点？
1. 如何写一个必然死锁的例子？
1. 发生死锁必须满足哪 4 个条件？
1. 如何用命令行和代码定位死锁？
1. 有哪些解决死锁问题的策略？
1. 讲一讲经典的哲学家就餐问题
1. final 的三种用法是什么？
1. 为什么加了 final 却依然无法拥有“不变性”？
1. 为什么 String 被设计为是不可变的？
1. 为什么需要 AQS？AQS 的作用和重要性是什么？
1. AQS 的内部原理是什么样的？
1. AQS 在 CountDownLatch 等类中的应用原理是什么？
