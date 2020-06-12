## Integer Cache

废话不多说----->直接上代码：

```
public class IntegerDemo {
    public static void main(String[] args) {
        Integer numA = 127;
        Integer numB = 127;

        Integer numC = 128;
        Integer numD = 128;

        System.out.println("numA == numB : "+ (numA == numB));
        System.out.println("numC == numD : "+ (numC == numD));
    }
}
```

结果：

```
numA == numB : true
numC == numD : false
```

​        What？这个输出结果怎么跟以往的认知有所出入呢？在我们的代码“Integer numA = 127”中，编译器会把基本数据的“自动装箱”(autoboxing)成包装类,所以这行代码就等价于“Integer numA = Integer.valueOf(127)”了,这样我们就可以进入valueOf方法查看它的实现原理。

````
  Integer类的源码
  
  private static class IntegerCache {
        static final int low = -128;
        static final int high = 127;
        static final Integer cache[];
  ......
  }
  
  public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
 }
````

​        从上面的源码可以看到，valueOf方法会先判断传进来的参数是否在IntegerCache的low与high之间，如果是的话就返回cache数组里面的缓存值,不是的话就new Integer(i)返回。

----------------------------------------------------------------------------------------------------------------------------------------

​       那我们再往上看一下IntegerCache，它是Integer的内部静态类，low默认是-128，high的值默认127，但是high可以通过JVM启动参数XX:AutoBoxCacheMax=size来修改（如图），如果我们按照这样修改了，然后再次执行上面代码，这时候2次输出都是true，因为缓存的区间变成-128~200了。

