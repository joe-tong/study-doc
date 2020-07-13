# Java有关强引用，软引用，弱引用，虚引用的记录



首先不要被一些对一些名词望而生畏，其实都是一些存在即合理的东西。

引用本身很好理解，引用类型的数据中存放的是实际对象的内存地址，垃圾回收时，就看对象是否存在引用。Java不需要开发人员分配内存和释放内存，但是可以通过四种引用类型来处理相关对象生命周期，配合jvm进行垃圾回收。





# 1.强引用

正常的创建对象，赋值变量都属于强引用类型，强引用类型在垃圾回收时，不会被回收，内存不足时直接抛出OutOfMemoryError错误。

```
byte[] data = new byte[2*1024*1024];
VM options:-Xms1m -Xmx1m -XX:+PrintGC
```

比如上面的示例，jvm指定最大堆内存1m，程序要创建一个2m的东西，程序运行时就会直接抛出OOM错误。当引用不再需要关联对象时，可以进行null赋值，方便jvm垃圾回收。





# 2.软引用(SoftReference)

具有软引用的对象，在内存足够时不会被回收，在发生OOM之前，才会被回收。在Java中使用SoftReference声明一个软引用，使用get方法返回对象的强引用，当软引用关联对象被回收时，get返回null。

留着有用，丢了也无妨，这种东东做缓存是合适的，内存不够用时可以被垃圾回收器回收，能够有效降低内存溢出风险。但注意软引用本身是一个强引用，同样需要清除，可以通过注册ReferenceQueue监听关联对象已被回收的软引用本身，进行清除操作。

```
byte[] data = new byte[1*1024*1024];
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
SoftReference<byte[]> softReference = new SoftReference<>(data,referenceQueue);
data = null;
System.out.println("before:"+softReference.get());

try {
    for (int i = 0; i < 10; i++) {
        byte[] temp = new byte[3*1024*1024];
        System.out.println("processing:"+softReference.get());
    }
} catch (Throwable t) {
    System.out.println("after:"+softReference.get());
    t.printStackTrace();
}
while(referenceQueue.poll()!=null){
    System.out.println("self:"+softReference);
    softReference.clear();
    softReference = null;
    System.out.println("last:"+softReference);
}
VM options:-Xms5m -Xmx5m -XX:+PrintGC
```





# 3.弱引用(WeakReference)

弱引用就更弱了，垃圾回收时直接会被回收掉，Java中使用WeakReference声明，一次gc就会被干掉，其余和软引用类似。

```
 byte[] data = new byte[1024*1024];
 WeakReference<byte[]> weakReference = new WeakReference<>(data);
 data = null;
 System.out.println("before:"+weakReference.get());
 System.gc();
 System.out.println("after:"+weakReference.get());
```

结果就是一次gc之后就拿不到关联对象了，注意这里是将data置为null之后，否则data是存在强引用关系的，软引用亦是如此。





# 4.虚引用(PhantomReference)

虚引用和没有引用一样，虚引用在使用上必须结合前面提到的ReferenceQueue，可以通过引用队列来监听是否关联对象将要被回收，可以在此时机进行处理操作，操作同软引用。

```
byte[] data = new byte[1024*1024];
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
PhantomReference<byte[]> phantomReference = new PhantomReference<> (data,referenceQueue);
System.out.println(phantomReference.get());
```

几种引用本质还是围绕在内存回收机制上，了解一些知识，有时候可能不会直接用在工作当中，但是可以在适当的地方或时机去优化部分程序，也能在阅读一些源码的时候起到帮助作用。