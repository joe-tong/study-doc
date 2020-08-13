#  有趣的设计——延时队列（DelayQueue）实现超时任务处理机制

**作者：星晴（当地小有名气，小到只有自己知道的杰伦粉）**

​       今天不得不吐槽一下老板了，我了去，又没发工资，这还让不让我活了，身负贷款，真的快要以贷养贷了。有没有搞错啊，老天啊；这句话憋了很久了，说出心声舒服多了，还是老老实实计算一下下个月怎么过吧！今天的互联网行情真是不好，我们公司也离倒闭不远了，希望慢慢能度过这段时间，有所好转，不然就得重新找工作了！！！

​      吐槽了这么多，还是回归正题，今天给大家分享一下我们项目中如何通过延时队列实现超时任务处理机制。

生产代码就不展示了，就一个Demo来玩吧

### DelayQueue

作用：根据执行时间进行排序，然后等待到执行时间，就能获取到相对的数据

应用场景：超时任务处理

​            

##### 1.创建DelayTask 实现Delayed

##### ![1596699988849](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596699988849.png)                              

属性说明：

- executeTime 延时任务的执行时间

- taskType 任务类型

- msg 具体执行的任务数据

方法：

- getDelay() 返回还剩多少时间执行: 通过任务执行时间减去当前时间

- compareTo() 返回排序大小： 队列之间的执行时间排序



##### 2.创建延迟队列处理器

![1596701665397](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596701665397.png)

说明：

- while循环获取队列

- delayQueue.take()，有数据就返回，没数据就等待 



##### 3.创建Test类

![test](C:\Users\Administrator\Pictures\test.png)

输出结果：

![1596702046505](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596702046505.png)

### 如果还有什么不懂，欢迎在下面留言！！！

关注公众号，有更多好玩的等着你！！！

![img](https://mmbiz.qpic.cn/mmbiz_jpg/YicpKkSXicfO23aLicEHTNZibc8zxtW31NSibuCibDgOk3UhJBq90Z1ibXdotRAzibukOAiaicYmWNZFm6R3YzolcOdbdE9Q/640?wx_fmt=jpeg)