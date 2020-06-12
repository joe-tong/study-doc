## java中的fail-fast(快速失败)机制

#### 简介

```
     fail-fast机制，即快速失败机制，是java集合中的一种错误检测机制。当在迭代集合的过程中对该集合的结构改变是，就有可能会发生fail-fast，即跑出ConcurrentModificationException异常。fail-fast机制并不保证在不同步的修改下一定抛出异常，它只是近最大努力去抛出，所以这种机制一般仅用于检测bug
```

#### fail-fast的出现场景

​        **在我们常见的java集合中就可能出现fail-fast机制，比如常见的ArrayList,HashMap.在多线程和单线程环境下都有可能出现快速失败**。

##### 1.单线程环境下的fail-fast例子：

```
    public static void main(String[] args) {
           List<String> list = new ArrayList<>();
           for (int i = 0 ; i < 10 ; i++ ) {
                list.add(i + "");
           }
           Iterator<String> iterator = list.iterator();
           int i = 0 ;
           while(iterator.hasNext()) {
                if (i == 3) {
                     list.remove(3);
                }
                System.out.println(iterator.next());
                i ++;
           }
     }

```

控制台打印：

```
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
	at java.util.ArrayList$Itr.next(ArrayList.java:851)
	at com.example.springboot_demo.fail_fast_safe.ArrayListFailFast.main(ArrayListFailFast.java:24)
```

​        该段代码定义了一个Arraylist集合，并使用迭代器遍历，在遍历过程中，刻意在某一步迭代中remove一个元素，这个时候，就会发生fail-fast 



##### 2.多线程环境下

```
public class ArrayListFailFastThreadPool {
    public static List<String> list = new ArrayList<>();

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            Iterator<String> iterator = list.iterator();
            while(iterator.hasNext()) {
                String s = iterator.next();
                System.out.println(this.getName() + ":" + s);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            super.run();
        }
    }

    private static class MyThread2 extends Thread {
        int i = 0;
        @Override
        public void run() {
            while (i < 10) {
                System.out.println("thread2:" + i);
                if (i == 2) {
                    list.remove(i);
                }
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                i ++;
            }
        }
    }

    public static void main(String[] args) {
        for(int i = 0 ; i < 10;i++){
            list.add(i+"");
        }
        MyThread1 thread1 = new MyThread1();
        MyThread2 thread2 = new MyThread2();
        thread1.setName("thread1");
        thread2.setName("thread2");
        thread1.start();
        thread2.start();
    }

}
```

控制台打印：

```
Exception in thread "thread1" java.util.ConcurrentModificationException
thread2:3
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
	at java.util.ArrayList$Itr.next(ArrayList.java:851)
	at com.example.springboot_demo.fail_fast_safe.ArrayListFailFastThreadPool$MyThread1.run(ArrayListFailFastThreadPool.java:20)
```

​      启动两个线程，分别对其中一个对list进行迭代，另一个在线程1的迭代过程中去remove一个元素，结果也是抛出了java.util.ConcurrentModificationException

#### Fail-fast原理

​     fail-fast是如何抛出ConcurrentModificationException异常的，又是在什么情况下才会抛出？

我们知道，对于集合入list，map类，我们都是可以通过迭代器来遍历，而Iterator其实只是一个接口，具体的实现还是具体的集合类的内部类实现Iterator并实现相关方法。这里我们就以ArrayList类为例。在ArrayList中，当调用list.iterator()时，其源码是：

```
 public Iterator<E> iterator() {
        return new Itr();
 }
 
 
  private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }
        
        //这段代码是关键
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

```

         可以看出，该方法才是判断是否抛出ConcurrentModificationException异常的关键。在该段代码中，当modCount != expectedModCount时，就会抛出该异常。很明显expectedModCount在整个迭代过程除了一开始赋予初始值modCount外，并没有再发生改变，所以可能发生改变的就只有modCount.
         在前面关于ArrayList扩容机制的分析中，可以知道在ArrayList进行add,remove,clear等涉及到修改集合中的元素个数的操作时，modCount就会发生改变（modCount++）所以当另一个线程(并发修改)或者同一个线程遍历过程中，调用相关方法使集合的个数发生改变，就会使modCount发生变化，这样在checkForComodification方法中就会抛出ConcurrentModificationException异常。
         类似的，hashMap中发生的原理也是一样的。

##### 避免fail-fast

***方法1***

​      在单线程的遍历过程中，如果要进行remove操作，可以调用迭代器的remove方法而不是集合类的remove方法

```
 public static void main(String[] args) {
           List<String> list = new ArrayList<>();
           for (int i = 0 ; i < 10 ; i++ ) {
                list.add(i + "");
           }
           Iterator<String> iterator = list.iterator();
           int i = 0 ;
           while(iterator.hasNext()) {
                if (i == 3) {
                     iterator.remove(); //迭代器的remove()方法
                }
                System.out.println(iterator.next());
                i ++;
           }
     }
```

**方法2**

      使用java并发包(java.util.concurrent)中的类来代替ArrayList 和hashMap。
​        CopyOnWriterArrayList代替ArrayList，CopyOnWriterArrayList在是使用上跟ArrayList几乎一样，CopyOnWriter是写时复制的容器(COW)，在读写时是线程安全的。该容器在对add和remove等操作时，并不是在原数组上进行修改，而是将原数组拷贝一份，在新数组上进行修改，待完成后，才将指向旧数组的引用指向新数组，所以对于CopyOnWriterArrayList在迭代过程并不会发生fail-fast现象。但 CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。

​       对于HashMap，可以使用ConcurrentHashMap，ConcurrentHashMap采用了锁机制，是线程安全的。在迭代方面，ConcurrentHashMap使用了一种不同的迭代方式。在这种迭代方式中，当iterator被创建后集合再发生改变就不再是抛出ConcurrentModificationException，取而代之的是在改变时new新的数据从而不影响原有的数据 ，iterator完成后再将头指针替换为新的数据 ，这样iterator线程可以使用原来老的数据，而写线程也可以并发的完成改变。即迭代不会发生fail-fast，但不保证获取的是最新的数据。

