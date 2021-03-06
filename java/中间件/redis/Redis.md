# Redis

## 1 、redis的数据结构

### （1） Redis数据结构之String

字符串类型是Redis中最为基础的数据存储类型，它在Redis中是二进制安全的，这便意味着该类型可以接受任何格式的数据，如JPEG图像数据或Json对象描述信息等。在Redis中字符串类型的Value最多可以容纳的数据长度是512M。

​	

#### 为何redis字符串是二进制安全？

在 C 言中，字符串可以用一个 \0 结尾的 char 数组来表示。

比如说， hello world 在 C 语言中就可以表示为 "hello world\0" 。

这种简单的字符串表示，在大多数情况下都能满足要求，但是，它并不能高效地支持长度计算和追加（append）这两种操作：

每次计算字符串长度（strlen(s)）的复杂度为 O(N)。

对字符串进行 N 次追加，必定需要对字符串进行 N 次内存重分配（realloc）。

而redis除了要处理c语言字符串之外，还需要处理redis的服务器协议等等。所以，redis实现的sds（简单动态字符串），是二进制安全的。

数据结构的定义如下：

```
struct sdshdr {  
    // buf 已占用长度  
    int len;  

   // buf 剩余可用长度  
    int free;  

   // 实际保存字符串数据的地方  
    char buf[];  
};
```

因为有了对字符串长度定义len, 所以在处理字符串时候不会以零值字节(\0)为字符串结尾标志.

二进制安全就是输入任何字节都能正确处理, 即使包含零值字节.



### （2）Redis数据结构之List

### （3）Redis数据结构之Hash

#### （4）Redis数据结构之Set

#### （5）Redis数据结构之SortedSet



## 2、redis的持久化机制

Redis是一个开源的高性能键值对数据库

是NoSQL技术阵营的一员

它通过提供多种键值数据类型来适应不同场景下的存储需求

借助一些高层级的接口使其可以胜任，如缓存、队列系统的不同角色

可用作缓存、队列、消息订阅/发布



#####    1). RDB持久化（redis系统默认持久化策略）：

​    该机制是指在指定的时间间隔内将内存中的数据集快照写入磁盘。   

可以通过配置设置自动做快照持久化的方式。我们可以配置redis在n秒内如果超过m个key被修改就自动做快照，下面是默认的快照保存配置

#####     2). AOF(append only file)持久化:

该机制将日志的形式记录服务器所处理的每一个写操作，在Redis服务器启动之初会读取该文件来重新构建数据库，以保证启动后数据库中的数据是完整的



### 3、mySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据

　　　redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略（回收策略）。

redis 提供 6种数据淘汰策略：

```
volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰

volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰

volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰

allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰

no-enviction（驱逐）：禁止驱逐数据
```

## 4、 redis常见性能问题和解决方案

```
　1).Master写内存快照，save命令调度rdbSave函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大的，会间断性暂停服务，所以Master最好不要写内存快照。

　2).Master AOF持久化，如果不重写AOF文件，这个持久化方式对性能的影响是最小的，但是AOF文件会不断增大，AOF文件过大会影响Master重启的恢复速度。Master最好不要做任何持久化工作，包括内存快照和AOF日志文件，特别是不要启用内存快照做持久化,如果数据比较关键，某个Slave开启AOF备份数据，策略为每秒同步一次。

　3).Master调用BGREWRITEAOF重写AOF文件，AOF在重写的时候会占大量的CPU和内存资源，导致服务load过高，出现短暂服务暂停现象。
　
　4). Redis主从复制的性能问题，为了主从复制的速度和连接的稳定性，Slave和Master最好在同一个局域网内

```



## 5、 redis的并发竞争问题如何解决?



Redis是单进程单线程的，redis利用队列技术将并发访问变为串行访问，消除了传统数据库串行控制的开销



## 8、redis是单线程，为什么那么快？

```
1、完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1);

2、数据结构简单，对数据操作也简单，Redis中的数据结构是专门进行设计的;

3、采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗;

4、使用多路I/O复用模型，非阻塞IO;

5、使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求;
```

以上几点都比较好理解，下边我们针对多路 I/O 复用模型进行简单的探讨：

多路 I/O 复用模型

​      多路I/O复用模型是利用 select、poll、epoll 可以同时监察多个流的 I/O 事件的能力，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有 I/O 事件时，就从阻塞态中唤醒，于是程序就会轮询一遍所有的流(epoll 是只轮询那些真正发出了事件的流)，并且只依次顺序的处理就绪的流，这种做法就避免了大量的无用操作。

　  这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程。采用多路 I/O 复用技术可以让单个线程高效的处理多个连接请求(尽量减少网络 IO 的时间消耗)，且 Redis 在内存中操作数据的速度非常快，也就是说内存内的操作不会成为影响Redis性能的瓶颈，主要由以上几点造就了 Redis 具有很高的吞吐量。



## 9、缓存穿透

​     缓存穿透是指查询一个一定不存在的数据，由于缓存是不命中时需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，造成缓存穿透。

解决办法：

 ```
1.布隆过滤
    对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃。还有最常见的则是采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。

2. 缓存空对象. 将 null 变成一个值.
    也可以采用一个更为简单粗暴的方法，如果一个查询返回的数据为空（不管是数 据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。
 ```

## 10、缓存雪崩

​    是指在我们设置缓存时采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到DB，DB瞬时压力过重雪崩。

解决方案：

```
    将系统中key的缓存失效时间均匀地错开，防止统一时间点有大量的key对应的缓存失效。比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。
```

## 11、缓存预热

```
1. nginx+lua将访问量上报到kafka中
要统计出来当前最新的实时的热数据是哪些，我们就得将商品详情页访问的请求对应的流量，日志，实时上报到kafka中，

2. storm从kafka中消费数据，实时统计出每个商品的访问次数，访问次数基于LRU内存数据结构的存储方案
优先用内存中的一个LRUMap去存放，性能高，而且没有外部依赖

否则的话，依赖redis,我们就是要防止reids挂掉数据丢失的情况，就不合适了；用mysql，扛不住高并发读写；用hbase，hadoop生态系统，维护麻烦，太重了，其实我们只要统计出一段时间访问最频繁的商品，然后对它们进行访问计数，同时维护出一个前N个访问最多的商品list即可

计算好每个task大致要存放的商品访问次数的数量，计算出大小，然后构建一个LURMap,apache commons collections有开源的实现，设定好map的最大大小，就会自动根据LRU算法去剔除多余的数据，保证内存使用限制，即使有部分数据被干掉了，然后下次来重新开始技术，也没什么关系，因为如果他被LRU算法干掉，那么它就不是热数据，说明最近一段时间很少访问，

3. 每个storm task启动的时候，基于zk分布式锁，将自己的id写入zk的一个节点中

4. 每个storm task负责完成自己这里的热数据的统计，比如每次计数过后，维护一个钱1000个商品的list，每次计算完都更新这个list

5. 写一个后台线程，每个一段时间，比如一分钟，将排名钱1000的热数据list,同步到zk中
```