# Fork/Join的妙处，你发现了吗？

## 1.什么是 Fork/Join 框架？

吐槽：不知道为什么很多人喜欢叫框架，不过是java多线程的一种方式

​        Fork/Join 框架是 Java7 提供了的一个用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后 得到大任务结果的框架。

​         Fork 就是把一个大任务切分为若干子任务并行的执行，Join 就是合并这些子任务的执行结果，最后得到这个大任务的结果。比如计 算1+2+.....＋10000，可以分割成 10 个子任务，每个子任务分别对 1000 个数进行求和，最终汇总这 10 个子任务的结果。如下图所示：

![image.png](https://i.loli.net/2020/12/28/IxADJNKTkS21FfY.png)

## Fork/Jion特性：

1. ForkJoinPool 不是为了替代 ExecutorService，而是它的补充，在某些应用场景下性能比 ExecutorService 更好。（见 Java Tip: When to use ForkJoinPool vs ExecutorService ）
2. ForkJoinPool 主要用于实现“分而治之”的算法，特别是分治之后递归调用的函数，例如 quick sort 等。
3.  ForkJoinPool 最适合的是计算密集型的任务，如果存在 I/O，线程间同步，sleep() 等会造成线程长时间阻塞的情况时，最好配合使用 ManagedBlocker。