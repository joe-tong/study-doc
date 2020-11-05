# Seata初使用——注册到nacos

参考网站：https://blog.csdn.net/weixin_43993065/article/details/105742596

### 文章目录

- [为什么要用分布式事务](https://blog.csdn.net/weixin_43993065/article/details/105742596#_5)
- [Seata简介](https://blog.csdn.net/weixin_43993065/article/details/105742596#Seata_8)
- [Seata能做啥](https://blog.csdn.net/weixin_43993065/article/details/105742596#Seata_11)
- - [分布式事务处理过程-ID+三组件模型](https://blog.csdn.net/weixin_43993065/article/details/105742596#ID_12)
  - [处理过程](https://blog.csdn.net/weixin_43993065/article/details/105742596#_18)
- [AT模式如何做到对业务的无侵入](https://blog.csdn.net/weixin_43993065/article/details/105742596#AT_33)
- - [是什么](https://blog.csdn.net/weixin_43993065/article/details/105742596#_34)
  - [一阶段加载](https://blog.csdn.net/weixin_43993065/article/details/105742596#_36)
  - [二阶段提交](https://blog.csdn.net/weixin_43993065/article/details/105742596#_43)
  - [三阶段回滚](https://blog.csdn.net/weixin_43993065/article/details/105742596#_47)
- [流程图](https://blog.csdn.net/weixin_43993065/article/details/105742596#_53)
- [使用方法](https://blog.csdn.net/weixin_43993065/article/details/105742596#_61)
- - [docker部署seata](https://blog.csdn.net/weixin_43993065/article/details/105742596#dockerseata_62)
  - [数据库建表](https://blog.csdn.net/weixin_43993065/article/details/105742596#_249)
  - - [在上面file.conf里指定的mysql中新建seata库，新建3张表](https://blog.csdn.net/weixin_43993065/article/details/105742596#fileconfmysqlseata3_250)
    - [给需要事务的库加个`undo_log`表](https://blog.csdn.net/weixin_43993065/article/details/105742596#undo_log_306)
  - [后端](https://blog.csdn.net/weixin_43993065/article/details/105742596#_322)
  - - [POM](https://blog.csdn.net/weixin_43993065/article/details/105742596#POM_323)
    - [配置文件](https://blog.csdn.net/weixin_43993065/article/details/105742596#_343)
    - [启动类](https://blog.csdn.net/weixin_43993065/article/details/105742596#_356)
    - [配置类](https://blog.csdn.net/weixin_43993065/article/details/105742596#_369)
    - [注解使用](https://blog.csdn.net/weixin_43993065/article/details/105742596#_400)
- [注意](https://blog.csdn.net/weixin_43993065/article/details/105742596#_404)

为什么要用分布式事务

单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用，分别使用三个独立的数据源,业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由本地事务来保证, 但是全局的数据一致性问题没法保证。

# Seata简介

Seata是一款开源的分布式事务解决方案,致力于在微服务架构下提供高性能和简单易用的分布式事务服务

# Seata能做啥

## 分布式事务处理过程-ID+三组件模型

- Transaction ID(XID)：全局唯一的事务id
- 三组件概念
  - Transaction Coordinator(TC)：事务协调器,维护全局事务的运行状态,负责协调并驱动全局事务的提交或回滚
  - Transaction Manager™：控制全局事务的边界,负责开启一个全局事务,并最终发起全局提交或全局回滚的决议
  - Resource Manager(RM)：控制分支事务,负责分支注册、状态汇报,并接受事务协调的指令,驱动分支(本地)事务的提交和回滚

## 处理过程

1. TM 向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一 的XID;
2. XID在微服务调用链路的上下文中传播;
3. RM向TC注册分支事务,将其纳入XID对应全局事务的管辖;
4. TM向TC发起针对XID的全局提交或回滚决议;
5. TC 调度XID下管辖的全部分支事务完成提交或回滚请求。

------

阶段版：

1. TM开启分布式事务(TM向TC注册全局事务记录)
2. 按业务场景,编排数据库、服务等事务内资源(RM向TC汇报资源准备状态)
3. TM结束分布式事务,事务一阶段结束(TM通知TC提交/回滚分布式事务)
4. TC汇报事务信息,决定分布式事务是提交还是回滚
5. TC通知所有RM提交/回滚资源,事务二阶段结束

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425000345884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

# AT模式如何做到对业务的无侵入

## 是什么

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425000911282.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

## 一阶段加载

在一阶段, Seata 会拦截“业务SQL" ,
1.解析SQL语义，找到“业务SQL"要更新的业务数据，在业务数据被更新前，将其保存成"before image” ,
2.执行“业务SQL"更新业务数据，在业务数据更新之后,
3.其保存成"after image”， 最后生成行锁。
以上操作全部在一个数据库事务内完成，这样保证了一阶段操作的原子性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425001117520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

## 二阶段提交

阶段如是顺利提交的话,
因为“业务SQL"在一阶段已经提交至数据库,所以Seata框架只需将一阶段保存的快照数据和行锁删掉，完成数据清理即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425001206569.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

## 三阶段回滚

二阶段回滚:
二阶段如果是回滚的话, Seata 就需要回滚一阶段已经执行的“业务SQL" ,还原业务数据。
回滚方式便是用"before image"还原业务数据;但在还原前要首先要校验脏写,对比“数据库当前业务数据”和"after image”,
如果两份数据完全-致就说明没有脏写， 可以还原业务数据，如果不一致就说明有脏写, 出现脏写就需要转人工处理。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425001302531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

# 流程图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425001340180.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

# 使用方法

## docker部署seata

1. `docker pull seataio/seata-server:1.0.0`
2. 在/home下新建seata文件夹，再在seata中新建config文件夹
   在config新建file.conf和registry.conf
   registry.conf:
   我是使用nacos注册中心，还有很多别的大家可以去瞅瞅官网

```
registry {
  type = "nacos"

  nacos {
    serverAddr = "nacos服务器IP:PORT"
    namespace = ""
    cluster = "default"
  }
}

config {
  type = "file"
  
  file {
    name = "file.conf"
  }
}
1234567891011121314151617
```

file.conf：这里主要是配置将规则数据库

```
# 主要是这两块，复制下面的全部文档可以根据这里的去搜索改
vgroup_mapping.fsp_tx_group = "default"
default.grouplist = "seata服务器IP:8091"

store {
  ## store mode: file、db
  mode = "db"
  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://数据库IP:3306/seata"
    user = "root"
    password = "123456"
    min-conn = 1
    max-conn = 10
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
  }
}
12345678910111213141516171819202122232425
transport {
  # tcp udt unix-domain-socket
  type = "TCP"
  #NIO NATIVE
  server = "NIO"
  #enable heartbeat
  heartbeat = true
  #thread factory for netty
  thread-factory {
    boss-thread-prefix = "NettyBoss"
    worker-thread-prefix = "NettyServerNIOWorker"
    server-executor-thread-prefix = "NettyServerBizHandler"
    share-boss-worker = false
    client-selector-thread-prefix = "NettyClientSelector"
    client-selector-thread-size = 1
    client-worker-thread-prefix = "NettyClientWorkerThread"
    # netty boss thread size,will not be used for UDT
    boss-thread-size = 1
    #auto default pin or 8
    worker-thread-size = 8
  }
  shutdown {
    # when destroy server, wait seconds
    wait = 3
  }
  serialization = "seata"
  compressor = "none"
}
service {
  #transaction service group mapping
  vgroup_mapping.fsp_tx_group = "default"
  #only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "IP:8091"
  #degrade, current not support
  enableDegrade = false
  #disable seata
  disableGlobalTransaction = false
}

client {
  rm {
    async.commit.buffer.limit = 10000
    lock {
      retry.internal = 10
      retry.times = 30
      retry.policy.branch-rollback-on-conflict = true
    }
    report.retry.count = 5
    table.meta.check.enable = false
    report.success.enable = true
  }
  tm {
    commit.retry.count = 5
    rollback.retry.count = 5
  }
  undo {
    data.validation = true
    log.serialization = "jackson"
    log.table = "undo_log"
  }
  log {
    exceptionRate = 100
  }
  support {
    # auto proxy the DataSource bean
    spring.datasource.autoproxy = false
  }
}

## transaction log store
store {
  ## store mode: file、db
  mode = "db"
  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://IP:3306/seata"
    user = "root"
    password = "123456"
    min-conn = 1
    max-conn = 10
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
  }
}
server {
  recovery {
    #schedule committing retry period in milliseconds
    committing-retry-period = 1000
    #schedule asyn committing retry period in milliseconds
    asyn-committing-retry-period = 1000
    #schedule rollbacking retry period in milliseconds
    rollbacking-retry-period = 1000
    #schedule timeout retry period in milliseconds
    timeout-retry-period = 1000
  }
  undo {
    log.save.days = 7
    #schedule delete expired undo_log in milliseconds
    log.delete.period = 86400000
  }
  #unit ms,s,m,h,d represents milliseconds, seconds, minutes, hours, days, default permanent
  max.commit.retry.timeout = "-1"
  max.rollback.retry.timeout = "-1"
}

## metrics settings
metrics {
  enabled = false
  registry-type = "compact"
  # multi exporters use comma divided
  exporter-list = "prometheus"
  exporter-prometheus-port = 9898
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120
```

1. 启动：
   因为是docker启动，nacos获取的是docker内网IP，所以一定要指定SEATA_IP，我当时整了好久

```shell
docker run --name seata-server \
-p 8091:8091 \
-e SEATA_IP=服务器IP \
-e SEATA_PORT=8091 \
-e SEATA_CONFIG_NAME=file:/root/seata-config/registry \
-v /home/seata/config:/root/seata-config  \
-d seataio/seata-server:1.0.0
1234567
```

## 数据库建表

### 在上面file.conf里指定的mysql中新建seata库，新建3张表

```sql
drop table if exists `global_table`;
create table `global_table` (
  `xid` varchar(128)  not null,
  `transaction_id` bigint,
  `status` tinyint not null,
  `application_id` varchar(32),
  `transaction_service_group` varchar(32),
  `transaction_name` varchar(128),
  `timeout` int,
  `begin_time` bigint,
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`xid`),
  key `idx_gmt_modified_status` (`gmt_modified`, `status`),
  key `idx_transaction_id` (`transaction_id`)
);


drop table if exists `branch_table`;
create table `branch_table` (
  `branch_id` bigint not null,
  `xid` varchar(128) not null,
  `transaction_id` bigint ,
  `resource_group_id` varchar(32),
  `resource_id` varchar(256) ,
  `lock_key` varchar(128) ,
  `branch_type` varchar(8) ,
  `status` tinyint,
  `client_id` varchar(64),
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`branch_id`),
  key `idx_xid` (`xid`)
);


drop table if exists `lock_table`;
create table `lock_table` (
  `row_key` varchar(128) not null,
  `xid` varchar(96),
  `transaction_id` long ,
  `branch_id` long,
  `resource_id` varchar(256) ,
  `table_name` varchar(32) ,
  `pk` varchar(36) ,
  `gmt_create` datetime ,
  `gmt_modified` datetime,
  primary key(`row_key`)
);
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051
```

### 给需要事务的库加个`undo_log`表

```sql
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
12345678910111213
```

## 后端

### POM

这里注意seata版本要跟我们服务器中安装的版本一致

```xml
<!--seata-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>seata-all</artifactId>
            <groupId>io.seata</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-all</artifactId>
    <version>1.0.0</version>
</dependency>
12345678910111213141516
```

### 配置文件

将上面写的file.conf和register.conf放到resources中

application.yml

```yml
spring:
  cloud:
    alibaba:
      seata:
        #自定义事务组名称需要与seata-server中的对应
        tx-service-group: fsp_tx_group
123456
```

### 启动类

```java
@EnableDiscoveryClient
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@MapperScan("com.boss.bes.user.permission.mapper")
public class UserPermissionApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserPermissionApplication.class, args);
    }
}
123456789
```

### 配置类

```java
@Configuration
public class DataSourceProxyConfig {

    @Value("${mybatis.mapperLocations}")
    private String mapperLocations;

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }

}
123456789101112131415161718192021222324252627
```

### 注解使用

给需要事务的方法打个`@GlobalTransactional`就好啦
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425002903139.png)官网有快速启动，大家可以去看一下。

# 注意

- seata不支持复合主键
  seata只支持单主键的表，且执行的sql中必须含有主键字段

例如由两张表的关联表，seata是不支持的 会报mutliply key
如果执行的sql中，不包含主键字段会报ShouldNeverHappenException： nullSeata初使用——注册到nacos

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[SsM4丶](https://me.csdn.net/weixin_43993065) 2020-04-25 00:30:45 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 1151 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) 收藏

分类专栏： [SpringCloudAlibaba](https://blog.csdn.net/weixin_43993065/category_9921730.html)

版权



### 文章目录

- [为什么要用分布式事务](https://blog.csdn.net/weixin_43993065/article/details/105742596#_5)
- [Seata简介](https://blog.csdn.net/weixin_43993065/article/details/105742596#Seata_8)
- [Seata能做啥](https://blog.csdn.net/weixin_43993065/article/details/105742596#Seata_11)
- - [分布式事务处理过程-ID+三组件模型](https://blog.csdn.net/weixin_43993065/article/details/105742596#ID_12)
  - [处理过程](https://blog.csdn.net/weixin_43993065/article/details/105742596#_18)
- [AT模式如何做到对业务的无侵入](https://blog.csdn.net/weixin_43993065/article/details/105742596#AT_33)
- - [是什么](https://blog.csdn.net/weixin_43993065/article/details/105742596#_34)
  - [一阶段加载](https://blog.csdn.net/weixin_43993065/article/details/105742596#_36)
  - [二阶段提交](https://blog.csdn.net/weixin_43993065/article/details/105742596#_43)
  - [三阶段回滚](https://blog.csdn.net/weixin_43993065/article/details/105742596#_47)
- [流程图](https://blog.csdn.net/weixin_43993065/article/details/105742596#_53)
- [使用方法](https://blog.csdn.net/weixin_43993065/article/details/105742596#_61)
- - [docker部署seata](https://blog.csdn.net/weixin_43993065/article/details/105742596#dockerseata_62)
  - [数据库建表](https://blog.csdn.net/weixin_43993065/article/details/105742596#_249)
  - - [在上面file.conf里指定的mysql中新建seata库，新建3张表](https://blog.csdn.net/weixin_43993065/article/details/105742596#fileconfmysqlseata3_250)
    - [给需要事务的库加个`undo_log`表](https://blog.csdn.net/weixin_43993065/article/details/105742596#undo_log_306)
  - [后端](https://blog.csdn.net/weixin_43993065/article/details/105742596#_322)
  - - [POM](https://blog.csdn.net/weixin_43993065/article/details/105742596#POM_323)
    - [配置文件](https://blog.csdn.net/weixin_43993065/article/details/105742596#_343)
    - [启动类](https://blog.csdn.net/weixin_43993065/article/details/105742596#_356)
    - [配置类](https://blog.csdn.net/weixin_43993065/article/details/105742596#_369)
    - [注解使用](https://blog.csdn.net/weixin_43993065/article/details/105742596#_400)
- [注意](https://blog.csdn.net/weixin_43993065/article/details/105742596#_404)



[官网](http://seata.io/zh-cn/)

# 为什么要用分布式事务

单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用，分别使用三个独立的数据源,业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由本地事务来保证, 但是全局的数据一致性问题没法保证。

# Seata简介

Seata是一款开源的分布式事务解决方案,致力于在微服务架构下提供高性能和简单易用的分布式事务服务

# Seata能做啥

## 分布式事务处理过程-ID+三组件模型

- Transaction ID(XID)：全局唯一的事务id
- 三组件概念
  - Transaction Coordinator(TC)：事务协调器,维护全局事务的运行状态,负责协调并驱动全局事务的提交或回滚
  - Transaction Manager™：控制全局事务的边界,负责开启一个全局事务,并最终发起全局提交或全局回滚的决议
  - Resource Manager(RM)：控制分支事务,负责分支注册、状态汇报,并接受事务协调的指令,驱动分支(本地)事务的提交和回滚

## 处理过程

1. TM 向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一 的XID;
2. XID在微服务调用链路的上下文中传播;
3. RM向TC注册分支事务,将其纳入XID对应全局事务的管辖;
4. TM向TC发起针对XID的全局提交或回滚决议;
5. TC 调度XID下管辖的全部分支事务完成提交或回滚请求。

------

阶段版：

1. TM开启分布式事务(TM向TC注册全局事务记录)
2. 按业务场景,编排数据库、服务等事务内资源(RM向TC汇报资源准备状态)
3. TM结束分布式事务,事务一阶段结束(TM通知TC提交/回滚分布式事务)
4. TC汇报事务信息,决定分布式事务是提交还是回滚
5. TC通知所有RM提交/回滚资源,事务二阶段结束

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425000345884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

# AT模式如何做到对业务的无侵入

## 是什么

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425000911282.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

## 一阶段加载

在一阶段, Seata 会拦截“业务SQL" ,
1.解析SQL语义，找到“业务SQL"要更新的业务数据，在业务数据被更新前，将其保存成"before image” ,
2.执行“业务SQL"更新业务数据，在业务数据更新之后,
3.其保存成"after image”， 最后生成行锁。
以上操作全部在一个数据库事务内完成，这样保证了一阶段操作的原子性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425001117520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

## 二阶段提交

阶段如是顺利提交的话,
因为“业务SQL"在一阶段已经提交至数据库,所以Seata框架只需将一阶段保存的快照数据和行锁删掉，完成数据清理即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425001206569.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

## 三阶段回滚

二阶段回滚:
二阶段如果是回滚的话, Seata 就需要回滚一阶段已经执行的“业务SQL" ,还原业务数据。
回滚方式便是用"before image"还原业务数据;但在还原前要首先要校验脏写,对比“数据库当前业务数据”和"after image”,
如果两份数据完全-致就说明没有脏写， 可以还原业务数据，如果不一致就说明有脏写, 出现脏写就需要转人工处理。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425001302531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

# 流程图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425001340180.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk5MzA2NQ==,size_16,color_FFFFFF,t_70)

# 使用方法

## docker部署seata

1. `docker pull seataio/seata-server:1.0.0`
2. 在/home下新建seata文件夹，再在seata中新建config文件夹
   在config新建file.conf和registry.conf
   registry.conf:
   我是使用nacos注册中心，还有很多别的大家可以去瞅瞅官网

```
registry {
  type = "nacos"

  nacos {
    serverAddr = "nacos服务器IP:PORT"
    namespace = ""
    cluster = "default"
  }
}

config {
  type = "file"
  
  file {
    name = "file.conf"
  }
}
1234567891011121314151617
```

file.conf：这里主要是配置将规则数据库

```
# 主要是这两块，复制下面的全部文档可以根据这里的去搜索改
vgroup_mapping.fsp_tx_group = "default"
default.grouplist = "seata服务器IP:8091"

store {
  ## store mode: file、db
  mode = "db"
  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://数据库IP:3306/seata"
    user = "root"
    password = "123456"
    min-conn = 1
    max-conn = 10
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
  }
}
12345678910111213141516171819202122232425
transport {
  # tcp udt unix-domain-socket
  type = "TCP"
  #NIO NATIVE
  server = "NIO"
  #enable heartbeat
  heartbeat = true
  #thread factory for netty
  thread-factory {
    boss-thread-prefix = "NettyBoss"
    worker-thread-prefix = "NettyServerNIOWorker"
    server-executor-thread-prefix = "NettyServerBizHandler"
    share-boss-worker = false
    client-selector-thread-prefix = "NettyClientSelector"
    client-selector-thread-size = 1
    client-worker-thread-prefix = "NettyClientWorkerThread"
    # netty boss thread size,will not be used for UDT
    boss-thread-size = 1
    #auto default pin or 8
    worker-thread-size = 8
  }
  shutdown {
    # when destroy server, wait seconds
    wait = 3
  }
  serialization = "seata"
  compressor = "none"
}
service {
  #transaction service group mapping
  vgroup_mapping.fsp_tx_group = "default"
  #only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "IP:8091"
  #degrade, current not support
  enableDegrade = false
  #disable seata
  disableGlobalTransaction = false
}

client {
  rm {
    async.commit.buffer.limit = 10000
    lock {
      retry.internal = 10
      retry.times = 30
      retry.policy.branch-rollback-on-conflict = true
    }
    report.retry.count = 5
    table.meta.check.enable = false
    report.success.enable = true
  }
  tm {
    commit.retry.count = 5
    rollback.retry.count = 5
  }
  undo {
    data.validation = true
    log.serialization = "jackson"
    log.table = "undo_log"
  }
  log {
    exceptionRate = 100
  }
  support {
    # auto proxy the DataSource bean
    spring.datasource.autoproxy = false
  }
}

## transaction log store
store {
  ## store mode: file、db
  mode = "db"
  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://IP:3306/seata"
    user = "root"
    password = "123456"
    min-conn = 1
    max-conn = 10
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
  }
}
server {
  recovery {
    #schedule committing retry period in milliseconds
    committing-retry-period = 1000
    #schedule asyn committing retry period in milliseconds
    asyn-committing-retry-period = 1000
    #schedule rollbacking retry period in milliseconds
    rollbacking-retry-period = 1000
    #schedule timeout retry period in milliseconds
    timeout-retry-period = 1000
  }
  undo {
    log.save.days = 7
    #schedule delete expired undo_log in milliseconds
    log.delete.period = 86400000
  }
  #unit ms,s,m,h,d represents milliseconds, seconds, minutes, hours, days, default permanent
  max.commit.retry.timeout = "-1"
  max.rollback.retry.timeout = "-1"
}

## metrics settings
metrics {
  enabled = false
  registry-type = "compact"
  # multi exporters use comma divided
  exporter-list = "prometheus"
  exporter-prometheus-port = 9898
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120
```

1. 启动：
   因为是docker启动，nacos获取的是docker内网IP，所以一定要指定SEATA_IP，我当时整了好久

```shell
docker run --name seata-server \
-p 8091:8091 \
-e SEATA_IP=服务器IP \
-e SEATA_PORT=8091 \
-e SEATA_CONFIG_NAME=file:/root/seata-config/registry \
-v /home/seata/config:/root/seata-config  \
-d seataio/seata-server:1.0.0
1234567
```

## 数据库建表

### 在上面file.conf里指定的mysql中新建seata库，新建3张表

```sql
drop table if exists `global_table`;
create table `global_table` (
  `xid` varchar(128)  not null,
  `transaction_id` bigint,
  `status` tinyint not null,
  `application_id` varchar(32),
  `transaction_service_group` varchar(32),
  `transaction_name` varchar(128),
  `timeout` int,
  `begin_time` bigint,
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`xid`),
  key `idx_gmt_modified_status` (`gmt_modified`, `status`),
  key `idx_transaction_id` (`transaction_id`)
);


drop table if exists `branch_table`;
create table `branch_table` (
  `branch_id` bigint not null,
  `xid` varchar(128) not null,
  `transaction_id` bigint ,
  `resource_group_id` varchar(32),
  `resource_id` varchar(256) ,
  `lock_key` varchar(128) ,
  `branch_type` varchar(8) ,
  `status` tinyint,
  `client_id` varchar(64),
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`branch_id`),
  key `idx_xid` (`xid`)
);


drop table if exists `lock_table`;
create table `lock_table` (
  `row_key` varchar(128) not null,
  `xid` varchar(96),
  `transaction_id` long ,
  `branch_id` long,
  `resource_id` varchar(256) ,
  `table_name` varchar(32) ,
  `pk` varchar(36) ,
  `gmt_create` datetime ,
  `gmt_modified` datetime,
  primary key(`row_key`)
);
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051
```

### 给需要事务的库加个`undo_log`表

```sql
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
12345678910111213
```

## 后端

### POM

这里注意seata版本要跟我们服务器中安装的版本一致

```xml
<!--seata-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>seata-all</artifactId>
            <groupId>io.seata</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-all</artifactId>
    <version>1.0.0</version>
</dependency>
12345678910111213141516
```

### 配置文件

将上面写的file.conf和register.conf放到resources中

application.yml

```yml
spring:
  cloud:
    alibaba:
      seata:
        #自定义事务组名称需要与seata-server中的对应
        tx-service-group: fsp_tx_group
123456
```

### 启动类

```java
@EnableDiscoveryClient
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@MapperScan("com.boss.bes.user.permission.mapper")
public class UserPermissionApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserPermissionApplication.class, args);
    }
}
123456789
```

### 配置类

```java
@Configuration
public class DataSourceProxyConfig {

    @Value("${mybatis.mapperLocations}")
    private String mapperLocations;

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }

}
123456789101112131415161718192021222324252627
```

### 注解使用

给需要事务的方法打个`@GlobalTransactional`就好啦
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425002903139.png)官网有快速启动，大家可以去看一下。

# 注意

- seata不支持复合主键
  seata只支持单主键的表，且执行的sql中必须含有主键字段

例如由两张表的关联表，seata是不支持的 会报mutliply key
如果执行的sql中，不包含主键字段会报ShouldNeverHappenException： null