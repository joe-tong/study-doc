# Spring

## 1.Spring特征

![1597302060682](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597302060682.png)

## 2.Spring核心组件



![1597302131737](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597302131737.png)

## 3.Spring 常用模块

![1597302219994](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597302219994.png)

## 4.**Spring** **常用注解** 

![1597302251701](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597302251701.png)

## 5.Spring第三方结合

![1597302286731](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597302286731.png)

## 6.Spring IOC 原理

### 6.1 概念

Spring 通过一个配置文件描述 Bean 及 Bean 之间的依赖关系，利用 Java 语言的反射功能实例化 Bean 并建立 Bean 之间的依赖关系

### 6.2 高层视图

![1597302421870](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597302421870.png)

### 6.3 实现

#### 6.3.1 BeanFactory-框架基础设施

​       BeanFactory 是 Spring 框架的基础设施，**面向 Spring 本身**；ApplicationContext 面向使用 **Spring 框架的开发者**，几乎所有的应用场合我们都直接使用 ApplicationContext 而非底层 的 BeanFactory。

![1597303057554](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597303057554.png)

##### 6.3.1.1 ListableBeanFactory

```
访问容器中 Bean 基本信息的若干方法，查看 Bean 的个数、获取某一类型 Bean 的配置名、查看容器中是否包括某一 Bean 等方法
```

##### 6.3.1.2 HierarchicalBeanFactory

```
父子级联 IoC 容器的接口，子容器可以通过接口方法访问父容器;
通过HierarchicalBeanFactory 接口， Spring 的 IoC 容器可以建立父子层级关联的容器体系，子
容器可以访问父容器中的 Bean，但父容器不能访问子容器的 Bean。Spring 使用父子容器实
现了很多功能，
```

##### 6.3.1.3 AutowireCapableBeanFactory: 自动装配

```
定义了将容器中的 Bean 按某种规则（如按名字匹配、按类型匹配等）进行自动装配的方法；
```

6.3.1.4 ConfigurableBeanFactory: 定制配置

```
是一个重要的接口，增强了 IoC 容器的可定制性，它定义了设置类装载器、属性编辑器、容器初始化后置处理器等方法；
```

##### 6.3.1.4 BeanDefinitionRegistry 注册表

##### 6.3.1.5 DefaultListableBeanFactory

##### 6.3.1.6 XmlBeanFactory

#### 6.3.2 ApplicationContext面向开发应用

​       ApplicationContext 由 BeanFactory 派 生 而 来 ， 提 供 了 更 多 面 向 实 际 应 用 的 功 能 。 ApplicationContext 继承了 HierarchicalBeanFactory 和 ListableBeanFactory 接口，在此基础 

上，还通过多个其他的接口扩展了 BeanFactory 的功能：

![1597304558723](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597304558723.png)

##### 6.3.2.1 ClassPathXmlApplicationContext：默认从类路径加载配置文件

### 6.4 Spring Bean 作用域

Spring 3 中为 Bean 定义了 5 中作用域，分别为 singleton（单例）、prototype（原型）、 

request、session 和 global session，5 种作用域说明如下： 

#### 6.4.1 singleton：单例模式（多线程下不安全）

#### 6.4.2 prototype:原型模式每次使用时创建

```
每次通过 Spring 容器获取 prototype 定义的 bean 时，容器都将创建
一个新的 Bean 实例，每个 Bean 实例都有自己的属性和状态
```

#### 6.4.3 Request：一次request 一个实例

```
在一次 Http 请求中，容器会返回该 Bean 的同一实例。而对不同的 Http 请求则会产生新的 Bean，而且该 bean 仅在当前 Http Request 内有效,当前 Http 请求结束，该 bean
实例也将会被销毁
```

#### 6.4.4 **session** 

```
在一次 Http Session 中，容器会返回该 Bean 的同一实例。而对不同的 Session 请
求则会创建新的实例，该 bean 实例仅在当前 Session 内有效。同 Http 请求相同，每一次
session 请求创建新的实例，而不同的实例之间不共享属性，且实例仅在自己的 session 请求
内有效，请求结束，则实例将被销毁
```

#### 6.4.5 global Session

```
在一个全局的 Http Session 中，容器会返回该 Bean 的同一个实例，仅在 使用 portlet context 时有效。
```

###  6.5.Spring Bean 生命周期

![1597305247252](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597305247252.png)

1、实例化

2、设置属性

3、执行setBeanName, BeanNameAware

4、执行setBeanFactory，BeanFactoryAware

5、执行setApplicationContext，ApplicationContext

6、执行postProcessorBeforeInitization方法，PostBeanProcessor接口

7、执行afterPropertiesSet方法，InitializingBean

8、执行自己定义的bean初始化方法、init-method

9、执行postProcessorAfterInilization方法、PostBeanProcessor接口

10、使用我们的Bean

11、执行destory，DisposableBean

12、调用定制的销毁方法，destory-method

### 6.6 Spring 依赖注入四种方式

6.6.1 构造器注入

6.6.2 setter 方法注入

6.6.3 静态工厂注入

6.6.4 实例工厂

### 6.7 5  种不同方式的自动装配

Spring 装配包括手动装配和自动装配，手动装配是有基于 xml 装配、构造方法、setter 方法等 

自动装配有五种自动装配的方式，可以用来指导 Spring 容器用自动装配方式来进行依赖注入。 

1. no：默认的方式是不进行自动装配，通过显式设置 ref 属性来进行装配。 

2. byName：通过参数名 自动装配，Spring 容器在配置文件中发现 bean 的 autowire 属性被设 

置成 byname，之后容器试图匹配、装配和该 bean 的属性具有相同名字的 bean。 

3. byType：通过参数类型自动装配，Spring 容器在配置文件中发现 bean 的 autowire 属性被 

设置成 byType，之后容器试图匹配、装配和该 bean 的属性具有相同类型的 bean。如果有多 

个 bean 符合条件，则抛出错误。 

4. constructor：这个方式类似于 byType， 但是要提供给构造器参数，如果没有确定的带参数 

的构造器参数类型，将会抛出异常。 

5. autodetect：首先尝试使用 constructor 来自动装配，如果无法工作，则使用 byType 方式。3



## 7. Spring APO 原理

![1597307575178](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597307575178.png)

### 7.1 概念

多个类的公共行为封装到一个可重用模块

### 7.2 AOP 两种代理方式

```
Spring 提供了两种方式来生成代理对象: JDKProxy 和 Cglib，具体使用哪种方式生成由
AopProxyFactory 根据 AdvisedSupport 对象的配置来决定。默认的策略是如果目标类是接口，
则使用 JDK 动态代理技术，否则使用 Cglib 来生成代理。
```

## 8.Spring MVC 原理

### 8.1 MVC流程

![1597307734591](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597307734591.png)

### 8.2 MVC常用注解

![1597308154073](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1597308154073.png)

