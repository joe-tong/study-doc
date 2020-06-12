# [Spring-data-jpa 学习笔记（二）](https://www.cnblogs.com/zeng1994/p/7627267.html)



​        通过上一篇笔记的，我们掌握了SpringData的相关概念及简单的用法。但上一篇笔记主要讲的是Dao层接口直接继承Repository接口，然后再自己定义方法。主要阐述了自定义方法时的一些规则及SpringData是如何来解析这些方法的。实际上，一些常用的方法SpringData已经帮我们定义好了，我们只需要定义Dao层接口时继承Repository的有相关功能子接口就ok了。本文主要讲的是Repository各个子接口有什么功能，了解了子接口的功能后，后续开发dao层就方便了。

​        这里再强调一下使用SpringData开发Dao层的步骤，这个步骤很关键。下面这段话是从上一篇笔记中copy过来的

**Spring Data JPA 进行持久层(即Dao)开发一般分三个步骤：**

- - 声明持久层的接口，该接口继承 Repository（或Repository的子接口，其中定义了一些常用的增删改查，以及分页相关的方法）。
  - 在接口中声明需要的业务方法。Spring Data 将根据给定的策略生成实现代码。
  - 在 Spring 配置文件中增加一行声明，让 Spring 为声明的接口创建代理对象。配置了 <jpa:repositories> 后，Spring 初始化容器时将会扫描 base-package 指定的包目录及其子目录，为继承 Repository 或其子接口的接口创建代理对象，并将代理对象注册为 Spring Bean，业务层便可以通过 Spring 自动封装的特性来直接使用该对象。