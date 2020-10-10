# MVCC多版本并发控制机制—包你学会

​        Mysq默认级别是可重复读隔离级别，那它是如何保证事务较高的隔离性。同样的sql查询语句在一个事务里多次执行查询结果相同，就算是其他事务对数据修改也不会影响当前事务sql语句对查询结果。

​       这个隔离性就是靠MVCC机制保证对，对一行数据的读和写两个操作默认是不会通过加锁互斥来保证隔离性，避免了频繁加锁互斥，而在串行化隔离级别为了保证较高的隔离性是通过将所有操作加锁互斥来实现的。

​       Mysql在读已提交和可重复读隔离级别下都实现了MVCC机制。



## MVCC机制如何实现的：

- Undo日志版本链
- Read view机制



## Undo日志版本链和read view机制详解

​       **Undo日志版本链**是指一行数据被多个事务依次修改过后，在每个事务修改完后，Mysql会保留修改前的数据

undo回滚日志，并且用两个隐藏字段trx_id和roll_pointer把这些undo日志串联起来形成一个历史记录版本链

​      当执行查询sql时会生成**一致性视图read-view**，它由执行查询时所有未提交事务id组（数组最小的id为min_id）和已创建的最大事务id（max_id）组成，查询的数据结果需要跟read-view做对比从而得到快照结果

#### 案例分析：

备注：案例分析很重要，一定要慢慢去看，不要心急哦

**版本链对比规则：**

1. 如果落在绿色部分（trx_id < min_id）,表示这个版本是已提交的事务生成的，这个数据是可见的
2. 如果落在红色部分（trx_id > max_id）,表示这个版本是由将来启动的事务生成的，是肯定不可见的
3. 如果落在黄色部分，那就是包括两种情况
   - 若row的trx_id在数组中，表示这个版本是由还没有提交的事务生成的，不可见，当前事务是可见的
   - 若row的trx_id不在数组中，表示这个版本是已经提交了的事务生成的，可见。

<a href="https://sm.ms/image/X39JFgr7wDudHs8" target="_blank"><img src="https://i.loli.net/2020/10/02/X39JFgr7wDudHs8.jpg" style="zoom:50%;"  ></a>

**根据这个版本链对比规则我们来假设几个场景进行分析**

场景一：

```
1.假设事务100、事务200、事务300开启 begin；最初的数据是trx_id = 60;
2.事务300进行 update account set name = lilei300 where id = 1;commit;
3.现在有一个sql查询，这个时候会生成一致性视图read-view；
readview：[100,200],300; [100,200]未提交的事务组ID，300表示已创建的最大事务ID；
```

<a href="https://sm.ms/image/8seFuEUWqGBlcbr" target="_blank"><img src="https://i.loli.net/2020/10/03/8seFuEUWqGBlcbr.jpg" style="zoom:50%;"  ></a>

```
我们分析日志版本链，从最新的日志开始，事务trx_id = 300;然后对照日志版本链规则，我们发现trx_id = 300落在黄色部分，然后trx_id不在数组中，说明是可见的，这个时候sql读取的name就是lilei300
```

场景二：

```
1.事务100执行update account set name = lilei1 where id = 1;update account set name = lilei2 where id = 1;
2.之前场景一session的再次查询，可重复读级别，同一个session一致性视图是不会创建新的，所以还是readview：[100,200],300;
```

<a href="https://sm.ms/image/lNj5VuzrTXFd6Ae" target="_blank"><img src="https://i.loli.net/2020/10/03/lNj5VuzrTXFd6Ae.jpg" style="zoom:50%;"  ></a>

我们分析日志版本链，从最新的日志开始，事务trx_id = 100;然后对照日志版本链规则,我们发现trx_id = 100落在黄色部分，然后trx_id 在数组里面，说明是不可见的，直到指向事务trx_id = 300;才发现事务是可见的，这个时候sql的数据name = lilei300 ，发现**同一个事物读取的数据一致**

场景三：

```
1.事务200执行update account set name = lilei3 where id = 1;update account set name = lilei4 where id = 1;事务100 commit；
2.新建一个session的查询，这个时候会创建一个新的readview：[200],300;
```

<a href="https://sm.ms/image/BdDmepoy4fSACtT" target="_blank"><img src="https://i.loli.net/2020/10/02/BdDmepoy4fSACtT.jpg" style="zoom:50%;"  ></a>



我们分析日志版本链，从最新的日志开始，事务trx_id = 200;然后对照日志版本链规则,我们发现trx_id = 200落在黄色部分，然后trx_id 在数组里面，说明是不可见的，直到指向事务trx_id = 100落到绿色部分，才发现事务是可见的；这个时候sql的数据name = lilei2；说明**新session一定读取到最新提交的数据**



#### 