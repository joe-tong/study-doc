## TreeSet源码分析

-----废话不多说先看一段代码

```
public class TreeSetTest {
    public static void main(String[] args) {
        TreeSet<Integer> treeSet = new TreeSet<>();
        treeSet.add(3);
        treeSet.add(1);
        treeSet.add(2);
        treeSet.add(4);
        System.out.println(treeSet);
    }
}
```

```
打印结果：
[1, 2, 3, 4]
```

​      通过结果我们发现TreeSet自动帮我们把传入的数据排序了，那底层源码是如何实现的呢？

##### 源码分析核心步骤

**1.TreeSet源码**

```
public class TreeSet{
    
  private transient NavigableMap<E,Object> m;
  private static final Object PRESENT = new Object();
    
  public TreeSet() {
        this(new TreeMap<E,Object>());
  } 
  
   public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }
}
```

​      通过这里的代码片段，我们发现treeSet的add(E e)的传入值其实保存到TreeMap的

key，而TreeMap的value只是一个Object,那我们看TreeMap的源码的put(key,value)

**2.TreeMap源码**

```
class TreeMap{

   public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
    
    
     static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;
     }
}
```

​       这里我们发现do{}while{}循环里面有个compareTo()不断比较，通过Entry的left和right进行设置顺序



总结：

1.TreeSet存储的数据不能重复，不然会被后添加的数据覆盖，应该值保存到TreeMap里面的key中

2.TreeSet存储的数据会自动排序，应为TreeMap的put方法里面有自动排序的功能

