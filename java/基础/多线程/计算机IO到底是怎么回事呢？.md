## 计算机I/O到底是怎么回事呢？

##### 1.在解释I/O之前，我们先了解什么是系统调用？

```
   进程想要获取磁盘上的数据，必须通过内核，因为能和硬件打交道的只能是内核，进程通知内核说我要磁盘中的数据，此过程就是系统调用。
```

##### 2.一次I/O的完成的步骤

```
   当进程发起系统调用时，这个系统调用就进入内核模式，然后开始I/O操作
I/O操作分为两个步骤：
1.磁盘把数据装载到内核的内存空间
2.内核的内存空间的数据copy到用户的内存空间中（此过程是I/O发生的地方）
```

**以下是进程获取数据的详细图解过程**

![205126317](D:\work_doc\205126317.png)

整过过程：一个进程需要对磁盘中的数据进行操作，则会向内核发起一个系统调用，然后此进程将会被切换出去，此进程会被挂起或者进入睡眠状态，也叫不可中断的睡眠，因为数据还没有得到，只有等到系统调用的结构完成后，则进程会被唤醒，继续接下来的操作，从系统调用开始到系统调用结束经过的步骤：

①进程向内核发起一个系统调用，

②内核接收到系统调用，知道是对文件的请求，于是告诉磁盘，把文件读取出来

③磁盘接收到来着内核的命令后，把文件载入到内核的内存空间里面

④内核的内存空间接收到数据之后，把数据copy到用户进程的内存空间(**此过程是****I/O****发生的地方**)

⑤进程内存空间得到数据后，给内核发送通知

⑥内核把接收到的通知回复给进程，此过程为唤醒进程，然后进程得到数据，进行下一步操作