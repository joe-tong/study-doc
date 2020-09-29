# Spring的IOC面试题

## 1.spring的ioc顶层视图(原理)

xml,javaConfig,@autowared --->解析到Spring容器内的BeanDefination表--->实例化Bean---> 放到spring缓存里面 --->getBean()

## 2.Spring里面对于IOC核心的几个类BeanFactory,ApplicationContext

- BeanFactory面向Spring的基础服务

  ```
  核心子类：
  1.DefaultListableBeanfactory 获取bean的基本信息
  2.AutowireCapableBeanFactory 自动装配Bean
  ```

- ApplicationContext面向开发这

  ```
  核心子类：
  1.AnnotationConfigApplicationContext
  2.ClassPathXmlApplicationContext
  ```

  ​

## 2. AnnotationConfigApplicationContext和ClassPathXmlApplicationContext（ 继承AbstractApplicationContext）的refresh方法

- obtainFreshBeanFactory 

  ```
  1.创建DefaultListableBeanFactory
  2.loadBeanDefination，把xml,javaconfig,注解,通过scaner,reader里面的AnnotationConfigUtils 解析成BeanDefinition，放到BeanDefinitionRegister里面
  ```

- invokeBeanFactoryPostProcessors 

  ```
  1.从Spring容器中找出BeanDefinitionRegisterPostProcessor和BeanFactoryPostProcessor的接口实现安顺序遍历
  2.ConfigurationClassPostProcessor优先级最高，do while 解析@Configuration和@Import
  1.ConfigurationClassPostProcessor把我们常见的注解@Configuration、@Import进行解析，解析完成之后把这些bean注册到BeanFactory中。
  ```

- registerBeanPostProcessors

  ```
  从Spring容器中找出的BeanPostProcessor接口的bean，并设置到BeanFactory的属性中。之后bean被实例化的时候会调用这个BeanPostProcessor。
  ```

- finishBeanFactoryInitialization

  ```
  实例化BeanFactory中已经被注册但是未实例化的所有实例

  getBean->doGetBean->getSingleton，三级缓存
  ```



## 3.Spring创建Bean的流程

首先需要了解是Spring它创建Bean的流程，我把它的大致调用栈绘图如下：

![Image text](https://img-blog.csdnimg.cn/20190619142513115.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Y2NDEzODU3MTI=,size_16,color_FFFFFF,t_70)

## 4.解决循环依赖

```
1.获取A对象
getBean（）, 判断singletonObjects里面是否有，有就返回Bean对象，没有就创建A对象(开始singletonObjects肯定没有A对象，所以一定会创建)
2.对A对象进行数据填充，发现依赖于对象B
3.获取对象B,递归调用getBean(),判断singletonObjects里面是否有，有就返回Bean对象，没有就创建A对象（）;
4.B数据填充，发现依赖于A,递归调用getBean()，发现singletonObjects里面有A，就返回A对象，B对象返回填充A的属性
```

