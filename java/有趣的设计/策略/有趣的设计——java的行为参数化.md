# 有趣的设计——java的行为参数化

 **作者：星晴（当地小有名气，小到只有自己知道的杰伦粉）**

​        今天公司离职了很多人，公司也没有安排什么事，我也不知道要干什么，每天在公司看看技术论坛，看看博客，写写文章打发时间吧，公司现在的氛围真的让人难受，希望过段时间有所好转吧。

​         今天跟大家分享一下java的行为参数化有什么好处，应用场景是什么？

## 前言

​         在软件工程中，众所周知，不管你做什么，用户的需求肯定会变。比方说餐厅采购人员去市场挑选鱼，当然鱼的选择有很多要求，比如**品种，重量**等等，反正就是食客想要什么，餐厅就会去这么做，毕竟做生意都是这样。     采购人员跟市场人员沟通后，市场人员就会统计有多少符合要求的鱼，那我们做的就是去筛选符合条件的鱼。

## 小试牛刀，筛选鱼

一般做法，首先创建一个鱼这个类，然后getSelectFish，遍历所有的鱼，如果有匹配的就放到数组里面，最后返回。以后有扩展，复制一份getSelectFish,里面的判断换掉就行了。

![1596766640372](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596766640372.png)

![1596767394456](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596767394456.png)

## 初步改进

把过滤参数放到方法签名里面，这样是不是就会减少复制的代码，但是请注意，如果在加一个筛选条件是不是又要复制一份，多一个参数。作为软件工程师怎么能打破DRY(do not repeat yourself)的软件工程原则呢？

![1596769829304](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596769829304.png)

## 行为参数化

行为参数化就是可以帮助处理频繁变更的需求的一种软件开发模式。

言以蔽之，它意味着拿出一个代码块，把它准备好却不去执行它。这个代码块以后可以被程序的其他部分调用，这意味着可以推迟这块代码的执行例如，可以将代码块作为参数传递给另一个方法，稍后再去执行它。这样，这个方法的行为就基于那块代码被参数化了。 

![1596770465308](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596770465308.png)

![1596770506404](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596770506404.png)

![1596770543932](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596770543932.png)

![1596770434696](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596770434696.png)

这样getSelectFish就可以适应所有的过滤条件，再也不用重复写getSelectFish了，当然你如果觉得这样就结束了，那你太没有追求了，我们如何优化代码呢？

## 优化代码

![1596770926037](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596770926037.png)

通过Lambda表达式可以把几个过滤类去掉，是不是更简单了，优化的路上我们要坚持不懈，如何可以更优化呢？

## 泛型进行通用

![1596771137705](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596771137705.png)

这样getSelectFish是不是更加通用了，可以适配所有的实物了



总结：如果遇到频繁变更的过滤条件，可以采用行为参数化进行设计。



关注公众号，有更多好玩的等着你！！！

![img](https://mmbiz.qpic.cn/mmbiz_jpg/YicpKkSXicfO23aLicEHTNZibc8zxtW31NSibuCibDgOk3UhJBq90Z1ibXdotRAzibukOAiaicYmWNZFm6R3YzolcOdbdE9Q/640?wx_fmt=jpeg)  