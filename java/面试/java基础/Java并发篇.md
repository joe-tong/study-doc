# Java并发篇

![juc](C:\Users\Administrator\Pictures\juc.png)

![1597134495777](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597134495777.png)

![1597134337350](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597134337350.png)

## 1. Java锁

### 1.1 乐观锁

### 1.2 悲观锁

### 1.3 自旋锁

### 1.4 Synchronized 同步锁

#### 1.4.1 核心组件

-  Wait Set：哪些调用 wait 方法被阻塞的线程被放置在这里；
-  Contention List：竞争队列，所有请求锁的线程首先被放在这个竞争队列中；
-  Entry List：Contention List 中那些有资格成为候选资源的线程被移动到 Entry List 中；
-  OnDeck：任意时刻，最多只有一个线程正在竞争锁资源，该线程被成为 OnDeck；
-  Owner：当前已经获取到所资源的线程被称为 Owner；
-  !Owner：当前释放锁的线程

#### 1.4.2  实现

![1597200964447](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597200964447.png)

### 1.5 ReentrantLock

### 1.6 Semaphore

### 1.7 锁的状态

锁状态总共有四种：无锁状态、偏向锁、轻量级锁和重量级锁

#### 1.7.1 轻量级锁

```
   “轻量级”是相对于使用操作系统互斥量来实现的传统锁而言的。但是，首先需要强调一点的是，
轻量级锁并不是用来代替重量级锁的，它的本意是在没有多线程竞争的前提下，减少传统的重量
级锁使用产生的性能消耗。在解释轻量级锁的执行过程之前，先明白一点，轻量级锁所适应的场
景是线程交替执行同步块的情况，如果存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀
为重量级锁。
```

#### 1.7.2 偏向锁

```
    Hotspot 的作者经过以往的研究发现大多数情况下锁不仅不存在多线程竞争，而且总是由同一线
程多次获得。偏向锁的目的是在某个线程获得锁之后，消除这个线程锁重入（CAS）的开销，看起
来让这个线程得到了偏护。引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级
锁执行路径，因为轻量级锁的获取及释放依赖多次 CAS 原子指令，而偏向锁只需要在置换
ThreadID 的时候依赖一次 CAS 原子指令（由于一旦出现多线程竞争的情况就必须撤销偏向锁，所
以偏向锁的撤销操作的性能损耗必须小于节省下来的 CAS 原子指令的性能消耗）。上面说过，轻
量级锁是为了在线程交替执行同步块时提高性能，而偏向锁则是在只有一个线程执行同步块时进
一步提高性能。
```

#### 1.7.3 重量级锁

```
    Synchronized 是通过对象内部的一个叫做监视器锁（monitor）来实现的。但是监视器锁本质又
是依赖于底层的操作系统的 Mutex Lock 来实现的。而操作系统实现线程之间的切换这就需要从用
户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么
Synchronized 效率低的原因。因此，这种依赖于操作系统 Mutex Lock 所实现的锁我们称之为
“重量级锁”。JDK 中对 Synchronized 做的种种优化，其核心都是为了减少这种重量级锁的使用。
JDK1.6 以后，为了减少获得锁和释放锁所带来的性能消耗，提高性能，引入了“轻量级锁”和
“偏向锁”。
```

### 1.8 锁升级

### 1.9 锁优化

- 减少锁持有时间：只用在有线程安全要求的程序上加锁

- 减小锁粒度：ConcurrentHashMap

- 锁分离：最常见的锁分离就是读写锁 ReadWriteLock

- 锁粗化：如果对同一个锁不停的进行请求、同步 和释放，其本身也会消耗系统宝贵的资源，反而不利于性能的优化。

  ```
  举例：
  for(int i=0;i<size;i++){
      synchronized(lock){
      }
  }
  需要锁粗化：
  synchronized(lock){
      for(int i=0;i<size;i++){
      }
  }
  ```

- 锁消除：锁消除是发生在编译器级别的一种锁优化方式。有时候我们写的代码完全不需要加锁，却执行了加锁操作。



## 2.线程

![1597213674116](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597213674116.png)

### 2.1 **线程基本方法** 

线程相关的基本方法有 wait，notify，notifyAll，sleep，join，yield 等

#### 2.1.1 线程等待wait

​       调用该方法的线程进入 **WAITING 状态**，只有等待另外线程的通知或被中断才会返回，需要注意的 

是调用 wait()方法后，**会释放对象的锁**。因此，wait 方法一般用在**同步方法或同步代码块中**。

#### 2.1.2  线程睡眠sleep

​       sleep 导致当前线程休眠，与 wait 方法不同的是 sleep 不会**释放当前占有的锁**,sleep(long)会导致 

线程进入 **TIMED-WATING 状态**，而 wait()方法会导致当前线程进入 **WATING 状态**

#### 2.1.3  线程让步yield

​        yield 会使当前线程**让出 CPU 执行时间片**，与其他线程一起重新竞争 CPU 时间片。一般情况下， 

优先级高的线程有更大的可能性成功竞争得到 CPU 时间片，但这又不是绝对的，有的操作系统对 

线程优先级并不敏感。

#### 2.1.4  线程中断interrupt

​       中断一个线程，其本意是给这个线程一个通知信号，会影响这个线程内部的一个中断标识位。这 

个线程本身并不会因此而改变状态(如阻塞，终止等)。 

#### 2.1.5 **Join** **等待其他线程终止** 

​       join() 方法，等待其他线程终止，在当前线程中调用一个线程的 join() 方法，则当前线程转为阻塞 

状态，回到另一个线程结束，当前线程再由阻塞状态变为就绪状态，等待 cpu 的宠幸。 

#### 2.1.6  线程唤醒 notify

​        Object 类中的 notify() 方法，唤醒在此对象监视器上等待的单个线程，如果所有线程都在此对象 

上等待，则会选择唤醒其中一个线程，选择是任意的，并在对实现做出决定时发生，线程通过调 

用其中一个 wait() 方法，在对象的监视器上等待，直到当前的线程放弃此对象上的锁定，才能继 

续执行被唤醒的线程，被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞 

争。类似的方法还有 notifyAll() ，唤醒再次监视器上等待的所有线程

### 2.2  线程上下文切换

​       巧妙地利用了时间片轮转的方式, CPU 给每个任务都服务一定的时间，然后把当前任务的状态保存 

下来，在加载下一任务的状态后，继续服务下一任务，任务的状态保存及再加载, 这段过程就叫做 

上下文切换。时间片轮转的方式使多个任务在同一颗 CPU 上执行变成了可能

![1597214670536](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597214670536.png)

#### 2.2.1 进程

指一个程序运行的实例。

#### 2.2.2 上下文

是指某一时间点 **CPU 寄存器和程序计数器的内容**

#### 2.2.3 寄存器

是 CPU 内部的数量较少但是速度很快的内存（与之对应的是 CPU 外部相对较慢的 RAM 主内 

存）。寄存器通过对常用值（通常是运算的中间值）的快速访问来提高计算机程序运行的速 

度。 

#### 2.2.4 程序计数器

是一个专用的寄存器，用于表明**指令序列中 CPU 正在执行的位置**，存的值为正在执行的指令 

的位置或者下一个将要被执行的指令的位置，具体依赖于特定的系统

## 3.线程池

### 3.1 线程池原理

### 3.2 线程池的组成

分为4个组成部分：

1. 线程池管理器：用于创建并管理线程池 

2. 工作线程：线程池中的线程 

3. 任务接口：每个任务必须实现的接口，用于工作线程调度其运行 

4. 任务队列：用于存放待处理的任务，提供一种缓冲机制

![1597219272759](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597219272759.png)



## 4.JAVA 阻塞队列

-  ArrayBlockingQueue ：由数组结构组成的有界阻塞队列。 

-  LinkedBlockingQueue ：由链表结构组成的有界阻塞队列。 

- PriorityBlockingQueue ：支持优先级排序的无界阻塞队列。 

  ```
  是一个支持优先级的无界队列。默认情况下元素采取自然顺序升序排列。可以自定义实现
  compareTo()方法来指定元素进行排序规则，或者初始化 PriorityBlockingQueue 时，指定构造
  参数 Comparator 来对元素进行排序。需要注意的是不能保证同优先级元素的顺序
  ```

-  DelayQueue：使用优先级队列实现的无界阻塞队列。 

  ```
  1.缓存系统的设计
  2.定时任务调度
  ```

-  SynchronousQueue：不存储元素的阻塞队列。 

  ```
      是一个不存储元素的阻塞队列。每一个 put 操作必须等待一个 take 操作，否则不能继续添加元素。
  SynchronousQueue 可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线
  程。队列本身并不存储任何元素，非常适合于传递性场景,比如在一个线程中使用的数据，传递给
  另 外 一 个 线 程 使 用 ， SynchronousQueue 的 吞 吐 量 高 于 LinkedBlockingQueue 和
  ArrayBlockingQueue。
  ```

  

-  LinkedTransferQueue：由链表结构组成的无界阻塞队列。 

-  LinkedBlockingDeque：由链表结构组成的双向阻塞队列

## 5.CyclicBarrier、CountDownLatch、Semaphore的用法

### 5.1 CountDownLatch（线程计数器 ）

CountDownLatch 类位于 java.util.concurrent 包下，利用它可以实现类似计数器的功能。比如有 

一个任务 A，它要等待其他 4 个任务执行完毕之后才能执行，此时就可以利用 CountDownLatch 

来实现这种功能了

```java
final CountDownLatch latch = new CountDownLatch(2);
     new Thread(){public void run() {
      System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
      Thread.sleep(3000);
      System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
      latch.countDown();
     };
         }.start();
      new Thread(){ public void run() {
      System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
      Thread.sleep(3000);
      System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
      latch.countDown();
    };
                  }.start();
      System.out.println("等待 2 个子线程执行完毕...");
      latch.await();
      System.out.println("2 个子线程已经执行完毕");
      System.out.println("继续执行主线程");
}
```

### 5.2 CyclicBarrier（回环栅栏-等待至 barrier 状态再全部同时执行）

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环 

是因为当所有等待线程都被释放以后，CyclicBarrier 可以被重用。我们暂且把这个状态就叫做 

barrier，当调用 await()方法之后，线程就处于 barrier 了。

### 5.3 Semaphore（信号量-控制同时访问的线程个数）

## 6.volatile(变量可见性、禁止重排序)

- 禁止重排序
- 变量可见性

![1597226618563](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597226618563.png)

当对非 volatile 变量进行读写的时候，每个线程先从内存拷贝变量到 CPU 缓存中。如果计算机有 

多个 CPU，每个线程可能在不同的 CPU 上被处理，这意味着每个线程可以拷贝到不同的 CPU  

cache 中。而声明变量是 volatile 的，JVM 保证了每次读变量都从内存中读，跳过 CPU cache  

这一步。

## 7.CAS

优点：解决synchronized的资源消耗过重

缺点：

- CPU开销较大
- 不能保证代码块的原子性
- ABA问题

## 8.AQS（同步器AbstractQueuedSynchronizer）

AQS 定义了一套多线程访问 共享资源的同步器框架