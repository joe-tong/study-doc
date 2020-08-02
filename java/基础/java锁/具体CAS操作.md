## 具体CAS操作

上一篇讲述了CAS机制,这篇讲解CAS具体操作.

什么是悲观锁、乐观锁？在java语言里，总有一些名词看语义跟本不明白是啥玩意儿，也就总有部分面试官拿着这样的词来忽悠面试者，以此来找优越感，其实理解清楚了，这些词也就唬不住人了。

- synchronized是悲观锁，这种线程一旦得到锁，其他需要锁的线程就挂起的情况就是悲观锁。
- CAS操作的就是乐观锁，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。

###### 那么问题来了，什么是CAS操作？

CAS是Compare-and-swap（比较与替换）的简写，是一种有名的无锁算法，在java中，我们主要分析Unsafe类，因为所有的CAS操作都是它来实现的，而在Unsafe类中这些方法也都是native方法

```
    public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

    public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

看到上面的解释是不是索然无味，查找了很多资料也没完全弄明白，通过几次验证后，终于明白，最终可以理解成一个无阻塞多线程争抢资源的模型。先上代码

```
package com.company.reentrantLock;

import java.util.concurrent.atomic.AtomicBoolean;
/**
 * Created by wxwall on 2017/6/2.
 */
public class AtomicBooleanTest implements Runnable{
    public static AtomicBoolean exits = new AtomicBoolean(true);
    public static void main(String[] args) {
        AtomicBooleanTest abd = new AtomicBooleanTest();
        Thread t1 = new Thread(abd);
        Thread t2 = new Thread(abd);
        t1.start();
        t2.start();
    }

    @Override
    public void run() {
        System.out.println("begin run");
        System.out.println("real " + exits.get());
        if(exits.compareAndSet(true,false)){
            System.out.println(Thread.currentThread().getName() + "  " + exits.get() );
            exits.set(true);
        }else{
            run();
        }
    }
}
```

输出结果：

```
begin run
real true
Thread-1  false
begin run
real true
Thread-0  false
```

这里无论怎么运行，Thread-1、Thread-0都会执行if=true条件，而且还不会产生线程脏读脏写，这是如何做到的了，这就用到了我们的compareAndSet(boolean expect,boolean update)方法，先上图简单讲解下程序原理，然后再分析compareAndSet作用。



![img](https:////upload-images.jianshu.io/upload_images/6073472-6720425e00244e92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/845/format/webp)

Paste_Image.png

这个图中重最要的是compareAndSet(true,false)方法要拆开成compare(true)方法和Set(false)方法理解，是compare(true)是等于true后，就马上设置共享内存为false，这个时候，其它线程无论怎么走都无法走到只有得到共享内存为true时的程序隔离方法区。
 但是这种得不到状态为true时使用递归算法是很耗cpu资源的，所以一般情况下，都会有线程sleep。

#### 总结

这篇文章并没有展开讲compareAndSet底层调用的是unsafe.compareAndSwapInt方法，因为这是native方法，很多人都会展开找源码，最后也只找到是调用CPU方法，没讲到具体用法，如果只用compareAndSet(true,false)举例则更加简单。
 这种无阻塞式的多线程操作数据，在大并发情况下，是一笔非常可观的性能提升，所以，如果在大并发或多线程性能要求高的情况下有更加好的技术选型，可以参考这种底层实现。















