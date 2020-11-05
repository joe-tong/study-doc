# UML类图还看不懂？

- **UML（Unified Modeling Language）**，是一种面向对象设计的建模工具，建模的核心是模型，模型是现实的简化、真实的抽象。

  在 UML 中，所有的描述包括：事务、关系、图这三部分构件组成，如下图为所有构件的关系。

  ![img](UML%E7%B1%BB%E5%9B%BE%E8%BF%98%E7%9C%8B%E4%B8%8D%E6%87%82%EF%BC%9F.assets/c2228e82-f590-45b6-a54c-62e98b602e2f.png)UML 构件关系图，来自设计模式

  **接下来**，我们就着重讲解UML中的类图关系，在乡村爱情人物里的体现。

  

  ### 1. 类图模型

  **UML 类图（Class Diagrams）**，是使用频率最高的 UML 图之一，类图可以表示出类、接口和它们之间的协作关系。各个接口、类、属性、方法，可以用如下方式表达。

  ![img](UML%E7%B1%BB%E5%9B%BE%E8%BF%98%E7%9C%8B%E4%B8%8D%E6%87%82%EF%BC%9F.assets/789999e1-08db-4811-97f7-f5dd681af2b5.png)UML 类图中，接口、类、属性、方法，表达方式

  

  ### 2. 继承关系

  **代码**

  ```
  public class 谢广坤 {
      private String 辈分;
      public void 作妖(){}
  }
  
  public class 谢永强 extends 谢广坤 {
  }
  
  public class 谢飞机 extends 谢广坤 {
  }
  ```

  **类图**

  ![img](UML%E7%B1%BB%E5%9B%BE%E8%BF%98%E7%9C%8B%E4%B8%8D%E6%87%82%EF%BC%9F.assets/22d020ea-741d-47ff-bf21-da68349e3b8b.png)UML类图，继承关系

  ------

  - 功能：继承关系
  - 概念：继承（Generaliztion）又叫泛化，用于表示子类继承父类的所有功能。
  - 场景：谢广坤的作妖技能，谢永强和谢飞机继承。谢飞机继承的更好，更能作。

  

  ### 3. 实现关系

  **代码**

  ```
  public interface 舞术 {
      void 招式();
  }
  
  public class 刘能 implements 舞术 {
      private String 来将姓名;
      public void 招式() {
      }
  }
  
  public class 赵四 implements 舞术 {
      private String 来将姓名;
      public void 招式() {
      }
  }
  ```

  **类图**

  ![img](UML%E7%B1%BB%E5%9B%BE%E8%BF%98%E7%9C%8B%E4%B8%8D%E6%87%82%EF%BC%9F.assets/6bf55ced-ee14-4214-9c71-ff594dea7032.png)UML类图，实现关系

  ------

  - 功能：实现关系
  - 概念：接口、抽象类声明的方法，由类实现（Realiztion）其功能。
  - 场景：在赵四和刘能的一场比舞中，赵四花式走位，被刘能找准时机一踢撂倒。

  

  ### 4. 组合关系

  **代码**

  ```
  public class 结婚证 {
  
      private 赵玉田 男方;
      private 刘英 女方;
  
      public void set男方(赵玉田 男方) {
          this.男方 = 男方;
      }
  
      public void set女方(刘英 女方) {
          this.女方 = 女方;
      }
  }
  
  public class 赵玉田 {
  
      private int 年龄;
      private String 性别;
  
  }
  
  public class 刘英 {
  
      private int 年龄;
      private int 性别;
  
  }
  ```

  **类图**

  ![img](UML%E7%B1%BB%E5%9B%BE%E8%BF%98%E7%9C%8B%E4%B8%8D%E6%87%82%EF%BC%9F.assets/f0ad96a4-d6ba-40bc-b4b9-1decf8b97270.png)UML类图，组合关系

  ------

  - 功能：组合关系
  - 概念：组合（Combination）关系表示类中整体与部分的关系，整体与部分相依相存。
  - 场景：赵玉田和刘英的结婚证，缺一不可。

  

  ### 5. 聚合关系

  **代码**

  ```
  public class 山庄 {
  
      private 宋晓峰 晓峰;
      private 李宝库 宝库;
  
      public void 药膳房(李宝库 宝库) {
          this.宝库 = 宝库;
      }
  
      public void 保安部(宋晓峰 晓峰) {
          this.晓峰 = 晓峰;
      }
  
  }
  
  public class 李宝库 {
  
      private String 职业;
  
  }
  
  public class 宋晓峰 {
  
      private String 职业;
  
  }
  ```

  **类图**

  ![img](UML%E7%B1%BB%E5%9B%BE%E8%BF%98%E7%9C%8B%E4%B8%8D%E6%87%82%EF%BC%9F.assets/7df20a6a-40c6-4fa1-930d-77136afd779a.png)UML类图，聚合关系

  ------

  - 功能：聚合关系
  - 概念：聚合（Aggregate）关系，也是用于表示对象的整体和部分，但成员对象可以与整体对象分离独立存在。
  - 场景：在⛰山庄中药膳方有李宝库、保安部有宋晓峰。但李宝库和宋晓峰都只是其中的一员，都可以离开山庄。

  

  ### 6. 关联关系

  **代码**

  ```
  public class 豆腐厂 {
      private 王小蒙 员工;
      public void 添加员工(王小蒙 小蒙){
          this.员工 = 小蒙;
      }
  }
  
  public class 王小蒙 {
      private 豆腐厂 企业;
      public void 添加企业(豆腐厂 豆腐厂){
          this.企业 = 豆腐厂;
      }
  
  }
  ```

  **类图**

  ![img](UML%E7%B1%BB%E5%9B%BE%E8%BF%98%E7%9C%8B%E4%B8%8D%E6%87%82%EF%BC%9F.assets/320ea36f-61ff-4e82-9499-3aa7c962386c.png)UML类图，关联关系

  ------

  - 功能：关联关系
  - 概念：关联（Association）关系，是类之间常用的一种关系，表示一类对象与另一类对象的联系。组合、聚合也属于这种关系，但关联关系更弱。
  - 场景：豆腐厂里有王小蒙，但豆腐厂里又不只是有王小蒙，还有王老七。即使小蒙不在，豆腐厂也可以正常运行。而王小蒙还有自己的其他企业，所以这属于一种关联关系。

  

  ### 7. 依赖关系

  **代码**

  ```
  public class 招商引资 {
      public void 招商(王大拿 大拿){
      }
  }
  
  public class 王大拿 {
      private String 资源;
  }
  ```

  **类图**

  ![img](UML%E7%B1%BB%E5%9B%BE%E8%BF%98%E7%9C%8B%E4%B8%8D%E6%87%82%EF%BC%9F.assets/aeb6d12b-c231-4cb2-8d2b-f1f71c23d86c.png)UML类图，依赖关系

  ------

  - 功能：依赖关系
  - 概念：依赖（Dependency）关系当表示一个事务需要使用另外一个事务时，可以使用依赖关系。
  - 场景：招商引资需要王大拿，但并只是就只有这一个大拿。王大拿不来，可能还有李大拿、张大拿。

  

  ## 四、赵家班全景类图

  **综上**，这6种关系里，组合、聚合、关联代码结构类似，可以从依赖的强弱进行理解。强弱关系依次是：继承 > 实现 > 组合 > 聚合 > 关联 > 依赖。

  为了更清楚的表达出 UML 类关系，我们把这些画到一整张图中，如下；

  ![img](UML%E7%B1%BB%E5%9B%BE%E8%BF%98%E7%9C%8B%E4%B8%8D%E6%87%82%EF%BC%9F.assets/ee866b04-98fe-48f1-9a26-a1acad7d3c01.png)UML类图，赵家班全景类图

  

  ## 五、总结

  - 有人说，如果我们和外星人👽非常友善的通信了。那么两个星球之间会进行一些交流，比如问，你好，地球人🌐人你多高呀？地球说1.75米。外星人晕了，米是什么单位？ **这样就只能选取两个星球通用的标准来定义，比如：1米是光在真空中1/299792458秒内经过的距离。**
  - 其实程序开发也是这样的，为了可以让大家减少对新知识内容的理解的沟通成本，需要定义一些沟通标准，比如UML类图。所以我们需要学习这些标准的工具化语言，来减少沟通成本，提升工作效率。
  - UML类图也是最常用的图稿，同时也非常易于掌握。为了可以把自己的知识面铺设的更加完善，技术栈掌握的更加夯实，也为了突破每一个阶段的瓶颈。那就需要不断学习，不断的积累，找机会破局。