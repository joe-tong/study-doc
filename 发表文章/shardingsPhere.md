# ShardingSphere你还不会吗？(第一篇)

**作者：星晴（当地小有名气，小到只有自己知道的杰伦粉）**

## 一.需求

​        我们做项目的时候，数据量比较大,单表千万级别的,需要分库分表，于是在网上搜索这方面的开源框架,最常见的就是mycat,sharding-sphere,最终我选择后者,用它来做分库分表比较容易上手。

### 二. 简介sharding-sphere

官网地址: https://shardingsphere.apache.org/

## 三.分库分表

### 3.1 pom.xml



```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<!--shardingsphere start-->
<!-- for spring boot -->
<dependency>
    <groupId>io.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>3.1.0</version>
</dependency>
<!-- for spring namespace -->
<dependency>
    <groupId>io.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-namespace</artifactId>
    <version>3.1.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

### 3.2 application.yml

```
# 数据源 cloud-db-0,cloud-db-1
sharding:
  jdbc:
    datasource:
      names: cloud-db-0,cloud-db-1
    # 第一个数据库
      cloud-db-0:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/cloud-db-0?characterEncoding=utf-8&&serverTimezone=GMT%2B8
        username: root
        password: 123456
    # 第二个数据库
      cloud-db-1:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/cloud-db-1?characterEncoding=utf-8&&serverTimezone=GMT%2B8
        username: root
        password: 123456

# 水平拆分的数据库（表） 配置分库 + 分表策略 行表达式分片策略
# 分库策略
    config:
      sharding:
        default-database-strategy:
          inline:
            sharding-column: id
            algorithm-expression: cloud-db-$->{id % 2}
        #分表策略 其中user为逻辑表 分表主要取决于age行
        tables:
           user:
             actual-data-nodes: cloud-db-$->{0..1}.user_$->{0..1}
             table-strategy:
               inline:
                 sharding-column: age
                 # 分片算法表达式
                 algorithm-expression: user_$->{age % 2}
      # 打印执行的数据库以及语句
      props:
        sql:
          show: true
```

### 3.3 数据库脚本

cloud-db-0:

```
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user_0
-- ----------------------------
DROP TABLE IF EXISTS `user_0`;
CREATE TABLE `user_0` (
  `id` int(11) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Table structure for user_1
-- ----------------------------
DROP TABLE IF EXISTS `user_1`;
CREATE TABLE `user_1` (
  `id` int(11) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

SET FOREIGN_KEY_CHECKS = 1;
```

cloud-db-1:

```
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user_0
-- ----------------------------
DROP TABLE IF EXISTS `user_0`;
CREATE TABLE `user_0` (
  `id` int(11) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Table structure for user_1
-- ----------------------------
DROP TABLE IF EXISTS `user_1`;
CREATE TABLE `user_1` (
  `id` int(11) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

SET FOREIGN_KEY_CHECKS = 1;
```

### 3.4 代码实现

User

```
@Data
@Entity
@Table(name = "user")
public class User {

    /**
     * 主键Id
     */
    @Id
    private int id;

    /**
     * 名称
     */
    private String name;

    /**
     * 年龄
     */
    private int age;
}
```

UserRepository

```
public interface UserRepository extends JpaRepository<User,Integer> {
}
```

UserServiceImpl

```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean save(User entity) {
        userRepository.save(entity);
        return true;
    }

    @Override
    public List<User> getUserList() {
        return userRepository.findAll();
    }

}
```

UserController

```
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/select")
    public List<User> select() {
        return userService.getUserList();
    }

    @GetMapping("/insert")
    public Boolean insert(User user) {
        return userService.save(user);
    }

}

```

### 3.5 测试

#### 1.分库、分表插入

```
http://localhost:8080/insert?id=1&name=lhd&age=12    
http://localhost:8080/insert?id=2&name=lhd&age=13    
http://localhost:8080/insert?id=3&name=lhd&age=14    
http://localhost:8080/insert?id=4&name=lhd&age=15
```

#### 2.分库、分表查询

```
http://localhost:8080/select
```

![](https://p.pstatp.com/origin/pgc-image/ee4fa204ab8f4f7d819a02ed246a9549)