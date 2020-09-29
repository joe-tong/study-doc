# MySQL事务的隔离性（MySQL事务实现原理补充）   

 **作者：星晴（当地小有名气，小到只有自己知道的杰伦粉）**

​        之前分享过一篇MySQL事务的实现原理，很多同学反映对事物的实现原理大致了解，但是觉得对事物的隔离性讲的太少了，没有完全弄懂，希望我能把这点讲透，为了满足大家的要求，特意写这篇文章，希望大家对事务的理解更加深层次。

​        之前我们讲过事务的有4大特点ACID，保证了事务的可靠性和并发处理。事务的可靠性就不再讲了，就是之前的redo log 和 undo log，事务如何在处理并发处理的时候，依旧能保证数据的一致性呢？这里就是通过隔离性去保证。那隔离性到底是什么，其实就是多个事务相互隔离，相互不影响，但是依旧能保证数据的正确性。那隔离性是如何做的呢？其实就是加锁，我们知道java里面就是通过加锁保证并发处理数据的正确性，其实mysql也是一样。如果多个事务对同一个数据处理，通过加锁，就可以保证事务等待处理数据。JAVA里有很多跟锁相关的类，那mysql有哪些锁呢？

希望大家有耐心的看完，并且跟着我实操一遍，才能更深刻的理解。

## 1.MySQL的锁分类

- 从性能上分为**乐观锁**（版本号对比来实现）和**悲观锁**

- 从多数据库操作的类型分为**读锁**和**写锁**（都属于据悲观锁）

  读锁(共享锁)：针对同一份数据，多个读操作可以同时进行而不会相互影响

  写锁(排它锁)：当前写操作没有完成前，它会阻断其他写锁和读锁

- 从数据操作的粒度分为**表锁**和**行锁**

### 1.1MySQL的表锁

```
每次操作锁住整张表。开销小，加锁快;不会出现死锁;锁定粒度大，发生锁冲
突的概率最高，并发度最低;
```

- 手动增加表锁
   lock table 表名称 read(write),表名称2 read(write);

- 查看表上加过的锁

   show open tables;

- 删除表锁

  unlock tables;

##### 表锁案例分析：

```
CREATE TABLE `mylock` (
`id` INT (11) NOT NULL AUTO_INCREMENT,
`NAME` VARCHAR (20) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE = MyISAM DEFAULT CHARSET = utf8;

INSERT INTO`test`.`mylock` (`id`, `NAME`) VALUES ('1', 'a');
INSERT INTO`test`.`mylock` (`id`, `NAME`) VALUES ('2', 'b');
INSERT INTO`test`.`mylock` (`id`, `NAME`) VALUES ('3', 'c');
INSERT INTO`test`.`mylock` (`id`, `NAME`) VALUES ('4', 'd');
```

###### 加表锁（读锁）

<a href="https://sm.ms/image/aOIPxNDHSeKBm4W" target="_blank"><img src="https://i.loli.net/2020/09/29/aOIPxNDHSeKBm4W.png" ></a>

当前session和其他session都可以读该表 

当前session中插入或者更新锁定的表都会报错，其他session插入或更新则会等 待

######  加表锁（写锁）

<a href="https://sm.ms/image/pM5f2eaR8EXw9Yn" target="_blank"><img src="https://i.loli.net/2020/09/29/pM5f2eaR8EXw9Yn.jpg" ></a>

当前session对该表的**增删改查**都没有问题，

其他session对该表的**所有操作**被阻 塞

###### 案例结论

简而言之，就是读锁会阻塞写，但是不会阻塞读。而写锁则会把读和写都阻塞。

### 1.2 MySQL的行锁

每次操作锁住一行数据。开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁 冲突的概率最低，并发度最高。

额外话题：

```
InnoDB与MYISAM的最大不同有两点：
1.支持事务（TRANSACTION）
2.支持行级锁
```

#### 1.2.1 行锁支持事务

```
MYISAM只有表锁，没有行锁，所以不支持事务；
索引才支持行锁；
```

##### 1.2.1.1事务（Transaction）及其ACID属性

-   原子性(Atomicity) ：事务是一个原子操作单元,其对数据的修改,要么全都执 行,要么全都不执行。
-  一致性(Consistent) ：在事务开始和完成时,数据都必须保持一致状态。这意 味着所有相关的数据规则都必须应用于事务的修改,以保持数据的完整性;事务结束 时,所有的内部数据结构(如B树索引或双向链表)也都必须是正确的。
-  隔离性(Isolation) ：数据库系统提供一定的隔离机制,保证事务在不受外部并 发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是 不可见的,反之亦然。
-  持久性(Durable) ：事务完成之后,它对于数据的修改是永久性的,即使出现系 统故障也能够保持。

##### 1.2.1.2并发事务处理带来的问题

- 更新丢失（Lost Update）: 当两个或多个事务**选择同一行**，然后基于最初选定的值更新该行时，由于每个事务都**不知道其他事务**的存在，就会发生丢失更新问题,最后的**更新覆盖**了由其他事务所做的更新。
-  脏读（Dirty Reads）: 一个事务正在对一条记录做修改，在这个事务完成并提交前，这条记录的数据就处于不一致的状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”数据，并据此作进一步的处理，就会产生未提 交的数据依赖关系。这种现象被形象的叫做“脏读”。 一句话：事务A读取到了事务B已经修改但尚未提交的数据，还在这个数据基础上做了操作。此时，如果B事务回滚，A读取的数据无效，不符合一致性要求。 
- 不可重读（Non-Repeatable Reads）: 一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现 其读出的数据已经发生了改变、或某些记录已经被删除了！这种现象就叫做“不 可重复读”。 一句话：事务A读取到了事务B已经提交的修改数据，不符合隔离性 
- 幻读（Phantom Reads）: 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插 入了满足其查询条件的新数据，这种现象就称为“幻读”。 一句话：事务A读取到了事务B提交的新增数据，不符合隔离性

##### 1.2.1.3隔离级别控制并发事务带来的问题

```
脏读”、“不可重复读”和“幻读”,其实都是数据库读一致性问题,必须由数据库提供一定的事务隔离机制来解决。
```

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| -------- | ---- | ---------- | ---- |
| 读未提交 | No   | No         | No   |
| 读已提交 | Yes  | No         | No   |
| 重复读   | Yes  | Yes        | No   |
| 串行化   | Yes  | Yes        | Yes  |

##### 1.2.1.4行锁与隔离级别案例分析

```
CREATE TABLE `account` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(255) DEFAULT NULL,
`balance` int(11) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
INSERT INTO `test`.`account` (`name`, `balance`) VALUES ('lilei', '450');
INSERT INTO `test`.`account` (`name`, `balance`) VALUES ('hanmei',
'16000');
INSERT INTO `test`.`account` (`name`, `balance`) VALUES ('lucy', '2400');
```

**行锁演示四种隔离级别以及会有什么问题**

###### 读未提交

问题：**脏读、不可重复读、幻读**

解决：无

- 打开一个客户端A和客户端B，并设置当前事务模式为read uncommitted（读未提交），查询表account的初始值:

  set tx_isolation='read-uncommitted';

  begin;

  select * from account;

  <a href="https://sm.ms/image/43hTHgIMEvpxnik" target="_blank"><img src="https://i.loli.net/2020/09/29/43hTHgIMEvpxnik.jpg" style="zoom:33%;"  ></a></a>

- 在客户端A的事务提交之前，打开另一个客户端B，更新表account, 并且不提交:

  update account set balance = balance - 50 where id = 1;

  <a href="https://sm.ms/image/OGcdW67tjsgk9qL" target="_blank"><img src="https://i.loli.net/2020/09/29/OGcdW67tjsgk9qL.jpg" style="zoom: 50%;"  ></a>

- 客户端A重新查询数据，能够发现读到B客户端未提交的事务，一旦客户端B的事务因为某种原因回滚，所有的操作都将会被撤销，那 客户端A查询到的数据其实就是**脏数据**

  select * from account;

  <a href="https://sm.ms/image/I5QKPdAgfl8Hpuj" target="_blank"><img src="https://i.loli.net/2020/09/29/I5QKPdAgfl8Hpuj.jpg" style="zoom: 67%;"  ></a>  

- 客户端B进行回滚，并且客户端A修改account表数据；

  rollback;

  <a href="https://sm.ms/image/XuBgHIzsADJYhOC" target="_blank"><img src="https://i.loli.net/2020/09/29/XuBgHIzsADJYhOC.jpg" style="zoom:67%;"  ></a>

  update account set balance = balance - 50 where id = 1;

   <a href="https://sm.ms/image/qwSvjE7zmgGuo65" target="_blank"><img src="https://i.loli.net/2020/09/29/qwSvjE7zmgGuo65.jpg" style="zoom:50%;"  ></a>

   ```
在客户端A执行更新语句update account set balance = balance - 50 where id =1，

lilei的balance没有变成350，居然是400，是不是很奇怪，数据不 一致啊;

如果你这么想就太天真 了，在应用程序代码中，我们可能会用400-50=350，然后set balance的值，并不知道其他会话回滚了，要想    解决这个问题可以采用读已提交的隔离级别
   ```

###### 读已提交

问题：不可重复读、幻读

解决：脏读

- 打开一个客户端A和B客户端，并设置当前事务模式为read committed（读已提交），查询表account的所有记录：

  set tx_isolation='read-committed';

  begin;

  select * from account;

  <a href="https://sm.ms/image/RqvaYGj4beCdzLH" target="_blank"><img src="https://i.loli.net/2020/09/29/RqvaYGj4beCdzLH.jpg" style="zoom:50%;"  ></a>

- 打卡客户端B进行修改account数据，并且不提交，客户端A没有读到客户端B未提交的数据，说明解决了**脏读**

  update account set balance = balance - 50 where id = 1;

  select * from account;

  <a href="https://sm.ms/image/wJ9I1GvA2ysd3S5" target="_blank"><img src="https://i.loli.net/2020/09/29/wJ9I1GvA2ysd3S5.jpg" style="zoom:50%;"  ></a>

  <a href="https://sm.ms/image/Wpis9Ubwho7BZnV" target="_blank"><img src="https://i.loli.net/2020/09/29/Wpis9Ubwho7BZnV.jpg" style="zoom:50%;"  ></a>

- 这个时候客户端B提交，客户端A再次查询，我们会发现客户端A在同一个事务里面读取的结果不一样（**不重复读**）

  commit;

  select * from account;

  <a href="https://sm.ms/image/89RJqwXrogIaLk3" target="_blank"><img src="https://i.loli.net/2020/09/29/89RJqwXrogIaLk3.jpg" style="zoom:50%;"  ></a>

###### 重复读

问题：幻读

解决：脏读、不可重复读

- 打开一个客户端A和B客户端，并设置当前事务模式为read repeated（重复读），查询表account的所有记录：

​      set tx_isolation='repeatable-read';

​      begin;

​      <a href="https://sm.ms/image/P6MTf7Wq1tbHRDx" target="_blank"><img src="https://i.loli.net/2020/09/29/P6MTf7Wq1tbHRDx.jpg" style="zoom:50%;"  ></a>

- 打卡客户端B进行修改account数据，客户端A在客户端B提交前和提交后查询数据,查询的数据一致，说明解决了**不可重复读**：

​       update account set balance = balance - 50 where id = 1;

​       commit;

​       select * from account;

​       <a href="https://sm.ms/image/B87aSk4rqhVyLxd" target="_blank"><img src="https://i.loli.net/2020/09/29/B87aSk4rqhVyLxd.jpg" style="zoom:50%;"  ></a>

​       <a href="https://sm.ms/image/MQguAdtFhoIW6SN" target="_blank"><img src="https://i.loli.net/2020/09/29/MQguAdtFhoIW6SN.jpg" style="zoom:50%;"  ></a>

​       可重复读的隔离级别下使用了MVCC(multi-version concurrency control)机制，select操作 不会更新版本号，是快照读（历史版本）；insert、update和delete会更新版本 号，是当前读（当前版本）。**MVCC等会精讲**

- 客户端B如果insert 一条数据，然后提交：

  insert into account values(4,'tpp',666);

  commit;

  <a href="https://sm.ms/image/VJqSa81lNz9wxRb" target="_blank"><img src="https://i.loli.net/2020/09/29/VJqSa81lNz9wxRb.jpg" style="zoom:50%;"  ></a>

- 客户端A查询数据，并没有查询到客户端B端插入到数据，这是因为可重复读原因：

   select * from account;

​       <a href="https://sm.ms/image/HS3to2qzRX8blZa" target="_blank"><img src="https://i.loli.net/2020/09/29/HS3to2qzRX8blZa.jpg" style="zoom:50%;"  ></a>

- 客户端A没有查询到，但是能够操作客户端B插入到数据吗？实际可以：

  update account set balance = 888 where id = 4;

  commit;

  select * from account;

  <a href="https://sm.ms/image/iSzQN7Mvg6lTaye" target="_blank"><img src="https://i.loli.net/2020/09/29/iSzQN7Mvg6lTaye.jpg" style="zoom:50%;"  ></a>

  从结果来看，我们发现客户端A没有查到客户端B插入的数据，但是却能修改客户端B插入的数据，这就是**幻读**

###### 串行化

问题：无

解决：脏读、不可重复读、幻读

- 打开一个客户端A和客户端B，并设置当前事务模式为read serializable（串行化），查询表account的初始值:

  set tx_isolation='serializable';

  begin;

  <a href="https://sm.ms/image/I4A9NzlHdk8eZGP" target="_blank"><img src="https://i.loli.net/2020/09/29/I4A9NzlHdk8eZGP.jpg" style="zoom:50%;"  ></a>

- 客户端A查询数据，并不提交：

  <a href="https://sm.ms/image/9SY7KBs2TFQovdu" target="_blank"><img src="https://i.loli.net/2020/09/29/9SY7KBs2TFQovdu.jpg" style="zoom:50%;"  ></a>

  

- 客户端B插入新的数据，发现报错：

  <a href="https://sm.ms/image/nHlL9QZ73T4zsvS" target="_blank"><img src="https://i.loli.net/2020/09/29/nHlL9QZ73T4zsvS.jpg" style="zoom:50%;"  ></a>

mysql中事务隔离级别为serializable时会锁表，因此 不会出现幻读的情况，这种隔离级别并发性极低，开发中很少会用到。

###### Mysql默认级别是repeatable-read，有办法解决幻读问题吗？

**间隙锁**

-  要避免幻读可以用间隙锁在客户端A下面执行update account set name = 'tong' where id > 10 and id <=20;

   set tx_isolation='repeatable-read';

   begin;

​        select * from account;

​       <a href="https://sm.ms/image/fJr4wOzsmiMCUkq" target="_blank"><img src="https://i.loli.net/2020/09/29/fJr4wOzsmiMCUkq.jpg" style="zoom:50%;"  ></a>

​      <a href="https://sm.ms/image/d3FjMlyiTWRxVcu" target="_blank"><img src="https://i.loli.net/2020/09/29/d3FjMlyiTWRxVcu.jpg" style="zoom:50%;"  ></a>

- 客户端B没法在这个范围所包含的 间隙里插入或修改任何数据

​       <a href="https://sm.ms/image/FGAYuj8ONeprmCa" target="_blank"><img src="https://i.loli.net/2020/09/29/FGAYuj8ONeprmCa.jpg" style="zoom:50%;"  ></a>

###### 无索引行锁会升级为表锁

- 锁主要是加在索引上，如果对非索引字段更新, 行锁可能会变表锁 客户端A执行：

​       update account set balance = 800 where name = 'lilei'; 

- 客户端B对该表任一行操作都会阻塞住 InnoDB的行锁是针对索引加的锁，不是针对记录加的锁。并且该索引不能失 效，否则都会从行锁升级为表锁。

###### 案例结论:

​       Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的 性能损耗可能比表级锁定会要更高一下，但是在整体并发处理能力方面要远远优 于MYISAM的表级锁定的。当系统并发量高的时候，Innodb的整体性能和 MYISAM相比就会有比较明显的优势了。



关注公众号，有更多好玩的等着你！！！

<a href="https://sm.ms/image/OXFgwdiTxkmSVl2" target="_blank"><img src="https://i.loli.net/2020/09/29/OXFgwdiTxkmSVl2.png" ></a>