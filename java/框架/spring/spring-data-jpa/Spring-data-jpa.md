## Spring-data-jpa

https://www.cnblogs.com/zeng1994/p/7575606.html

**1.spring-data-jpa的简单介绍**

```
SpringData : Spring 的一个子项目。用于简化数据库访问，支持NoSQL 和 关系数据存储。其主要目标是使数据库的访问变得方便快捷。

SpringData 项目所支持 NoSQL 存储：

 MongoDB （文档数据库）
 Neo4j（图形数据库）
 Redis（键/值存储）
 Hbase（列族数据库）
SpringData 项目所支持的关系数据存储技术：

JDBC
JPA


JPA Spring Data ： 致力于减少数据访问层 (DAO) 的开发量， 开发者唯一要做的就只是声明持久层的接口，其他都交给 Spring Data JPA 来帮你完成！

框架怎么可能代替开发者实现业务逻辑呢？比如：当有一个 UserDao.findUserById() 这样一个方法声明，大致应该能判断出这是根据给定条件的 ID 查询出满足条件的 User 对象。Spring Data JPA 做的便是规范方法的名字，根据符合规范的名字来确定方法需要实现什么样的逻辑。
```

**2.Spring Data JPA 进行持久层(即Dao)开发一般分三个步骤：**

- - 声明持久层的接口，该接口继承 Repository（或Repository的子接口，其中定义了一些常用的增删改查，以及分页相关的方法）。
  - 在接口中声明需要的业务方法。Spring Data 将根据给定的策略生成实现代码。
  - 在 Spring 配置文件中增加一行声明，让 Spring 为声明的接口创建代理对象。配置了 <jpa:repositories> 后，Spring 初始化容器时将会扫描 base-package 指定的包目录及其子目录，为继承 Repository 或其子接口的接口创建代理对象，并将代理对象注册为 Spring Bean，业务层便可以通过 Spring 自动封装的特性来直接使用该对象。



**3.spring-data-jpa的配置请看博客**

**4.在dao层声明接口**

```
在dao层声明接口
        在框架整合完成后，我们就可以开始使用SpringData了，在（3）中我们新建了一个Person实体类，我们就利用这个Person类来展开讲解。
        使用SpringData后，我们只需要在com.zxy.dao层声明接口，接口中定义我们要的方法，且接口继承Repository接口或者是Repository的子接口，这样就可以操作数据库了。但是在接口中定义的方法是要符合一定的规则的，这个规则在后面会讲到。其实我们也可以写接口的实现类，这个在后续也会讲解。
        先新建一个名为PersonDao的接口，它继承Repository接口；继承Repository接口的时候那两个泛型需要指定具体的java类型。第一个泛型是写实体类的类型，这里是Person；第二个泛型是主键的类型，这里是Integer。 在这个接口中定义一个叫做getById(Integer id)的方法，然后我们后面在调用这个方法测试下。
```

**5.SpringData方法定义规范**

````
（1）简单的条件查询的方法定义规范
方法定义规范如下：
简单条件查询：查询某一个实体或者集合
按照SpringData规范，查询方法于find|read|get开头，涉及条件查询时，条件的属性用条件关键字连接，要注意的是：属性首字母需要大写。
支持属性的级联查询；若当前类有符合条件的属性， 则优先使用， 而不使用级联属性。 若需要使用级联属性， 则属性之间使用 _ 进行连接。
        
下面来看个案例吧，操作的实体依旧上面的Person，下面写个通过id和name查询出Person对象的案例。
在PersonDao这个接口中，定义一个通过id和name查询的方法
 
// 通过id和name查询实体,sql:  select * from jpa_persons where id = ? and name = ?
Person findByIdAndName(Integer id, String name);
        在TestQucikStart这个测试类中，写个单元测试方法testFindByIdAndName来测试这个dao层的方法是否可用
 
/** 测试getByIdAndName方法 */
@Test
public void testGetByIdAndName() {
    PersonDao personDao = ctx.getBean(PersonDao.class);
    Person person = personDao.findByIdAndName(1, "test0");
    System.out.println(person);
}
     
        
````

##### （2）支持的关键字

​        直接在接口中定义方法，如果符合规范，则不用写实现。目前支持的关键字写法如下：

​        ![img](https://images2018.cnblogs.com/blog/1222688/201807/1222688-20180711202018082-2115608978.png)

​        ![img](https://images2018.cnblogs.com/blog/1222688/201807/1222688-20180711202018967-488914275.png)

##### （3）一个属性级联查询的案例

​        Dao层接口中定义的方法支持级联查询，下面通过一个案例来介绍这个级联查询：

- - 在com.zxy.entity包下新建一个Address的实体，代码如下图，setter和getter方法我省略了

​                    ![img](https://images2018.cnblogs.com/blog/1222688/201807/1222688-20180711202022893-1916087965.png)

- - 修改Person类，添加address属性，使Person和Address成多对一的关系，设置外键列名为address_id ，添加的代码如下图：

##### （4）查询方法解析流程

​        通过以上的学习，掌握了在接口中定义方法的规则，我们就可以定义出很多不用写实现的方法了。这里再介绍下查询方法的解析的流程吧，掌握了这个流程，对于定义方法有更深的理解。

​        **<1>** 方法参数不带特殊参数的查询

​        假如创建如下的查询：findByUserDepUuid()，框架在解析该方法时，流程如下：

- - 首先剔除 findBy，然后对剩下的属性进行解析，假设查询实体为Doc
  - 先判断 userDepUuid（根据 POJO 规范，首字母变为小写）是否为查询实体的一个属性，如果是，则表示根据该属性进行查询；如果没有该属性，继续往下走
  - 从右往左截取第一个大写字母开头的字符串(此处为Uuid)，然后检查**剩下的字符串**是否为查询实体的一个属性，如果是，则表示根据该属性进行查询；如果没有该属性，则重复这一步，继续从右往左截取；最后假设 user 为查询实体的一个属性
  - 接着处理剩下部分（DepUuid），先判断 user 所对应的类型是否有depUuid属性，如果有，则表示该方法最终是根据 "Doc.user.depUuid" 的取值进行查询；否则继续按照步骤3的规则从右往左截取，最终表示根据 "Doc.user.dep.uuid" 的值进行查询。

​        可能会存在一种特殊情况，比如 Doc包含一个 user 的属性，也有一个 userDep 属性，此时会存在混淆。可以

明确在级联的属性之间加上 "_"

以显式表达意图，比如

"findByUser_DepUuid()"或者"findByUserDep_uuid()"。

 **<2>** 方法参数带特殊参数的查询

​         特殊的参数： 还可以直接在方法的参数上加入分页或排序的参数，比如：

​         Page<UserModel> findByName(String name, Pageable pageable)

​         List<UserModel> findByName(String name, Sort sort);

## 四、@Query注解

```
通过上面的学习，我们在dao层接口按照规则来定义方法就可以不用写方法的实现也能操作数据库。但是如果一个条件查询有多个条件时，写出来的方法名字就太长了，所以我们就想着不按规则来定义方法名。我们可以使用@Query这个注解来实现这个功能，在定义的方法上加上@Query这个注解，将查询语句声明在注解中，也可以查询到数据库的数据。
（1）使用Query结合jpql语句实现自定义查询
在PersonDao接口中声明方法，放上面加上Query注解，注解里面写jpql语句，代码如下：

 
// 自定义的查询,直接写jpql语句; 查询id<? 或者 名字 like?的person集合
@Query("from Person where id < ?1 or name like ?2")
List<Person> testPerson(Integer id, String name);
    
// 自定义查询之子查询,直接写jpql语句; 查询出id最大的person
@Query("from Person where id = (select max(p.id) from Person as p)")
Person testSubquery();
在TestQuickStart中添加以下代码，测试dao层中使用Query注解的方法是否可用
 
/** 测试用Query注解自定义的方法  */
@Test
public void testCustomMethod() {
    PersonDao personDao = ctx.getBean(PersonDao.class);
    List<Person> list = personDao.testPerson(2, "%admin%");
    for (Person person : list) {
        System.out.println("查询结果： name=" + person.getName() + ",id=" + person.getId());
    }
    System.out.println("===============分割线===============");
    Person person = personDao.testSubquery();
    System.out.println("查询结果： name=" + person.getName() + ",id=" + person.getId());

}
```

##### （2）索引参数和命名参数

​        在写jpql语句时，查询条件的参数的表示有以下2种方式：

- - 索引参数方式如下图所示，索引值从**1**开始，查询中**'?x'**的个数要和方法的参数个数一致，且顺序也要一致

​                    ![img](https://images2018.cnblogs.com/blog/1222688/201807/1222688-20180711202027267-1210799300.png)

- - 命名参数方式（推荐使用这种方式）如下图所示，可以用**':参数名'**的形式，在方法参数中使用@Param（"参数名"）注解，这样就可以不用按顺序来定义形参



​        说一个**特殊情况**，那就是自定义的Query查询中jpql语句有like查询时，可以直接把%号写在参数的前后，这样传参数就不用把%号拼接进去了。使用案例如下，调用该方法时传递的参数直接传就ok。

​               ![img](https://images2018.cnblogs.com/blog/1222688/201807/1222688-20180711202028077-1363230092.png)



##### （3）使用@Query来指定使用本地SQL查询

​        如果你不熟悉jpql语句，你也可以写sql语句查询，只需要在@Query注解中设置nativeQuery=true。直接来看案例吧

- - dao层接口写法如下图所示

​                    ![img](https://images2018.cnblogs.com/blog/1222688/201807/1222688-20180711202028529-1039349537.png)

## 五、@Modifying注解和事务

## 五、@Modifying注解和事务

##### （1）@Modifying注解的使用

​        @Query与@Modifying这两个注解一起使用时，可实现个性化更新操作及删除操作；例如只涉及某些字段更新时最为常见。

下面演示一个案例，把id小于3的person的name都改为'admin'

- - dao层代码如下所示

//可以通过自定义的 JPQL 完成 UPDATE 和 DELETE 操作. 注意: JPQL 不支持使用 INSERT

//在 @Query 注解中编写 JPQL 语句, 但必须使用 @Modifying 进行修饰. 以通知 SpringData, 这是一个 UPDATE 或 DELETE 操作

//UPDATE 或 DELETE 操作需要使用事务, 此时需要定义 Service 层. 在 Service 层的方法上添加事务操作. 

//默认情况下, SpringData 的每个方法上有事务, 但都是一个只读事务. 他们不能完成修改操作!

@Modifying

@Query("UPDATE Person p SET p.name = :name WHERE p.id < :id")

int updatePersonById(@Param("id")Integer id, @Param("name")String updateName);

- 由于这个更新操作，只读事务是不能实现的，因此新建PersonService类，在Service方法中添加事务注解。PersonService的代码如下图所示

- - 测试类中直接调用service的方法就ok了，测试代码如下图

​                ![img](https://images2018.cnblogs.com/blog/1222688/201807/1222688-20180711202029496-1091828234.png)

​        使用@Modifying+@Query时的**注意事项**：

- - - 方法返回值是int，表示影响的行数
    - 在调用的地方必须加事务，没事务不执行

##### （2）事务

- - Spring Data 提供了默认的事务处理方式，即所有的查询均声明为只读事务。
  - 对于自定义的方法，如需改变 Spring Data 提供的事务默认方式，可以在方法上注解 @Transactional 声明 
  - 进行多个 Repository 操作时，也应该使它们在同一个事务中处理，按照分层架构的思想，这部分属于业务逻辑层，因此，需要在 Service 层实现对多个 Repository 的调用，并在相应的方法上声明事务。

