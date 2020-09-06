# @Transaction失效场景

## 1.@Transactional 应用在非 public 修饰的方法上

```
原因：
SpringAop代理模式实现的，里面有个类AbstractFallbackTransactionAttributeSource下的一个方法computeTransactionAttribute的对是否是public进行校验了

```

```
protected TransactionAttribute computeTransactionAttribute(Method method,
    Class<?> targetClass) {
        // Don't allow no-public methods as required.
        if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
       return null;
}
```

## 2.@Transactional 注解属性 propagation 设置错误

```
1.PROPAGATION_SUPPORTS
2.PROPAGATION_NOT_SUPPORTED
3.PROPAGATION_NEVER
```

## 3、@Transactional 注解属性 rollbackFor 设置错误

```
未检出到异常
```

## 4.同一个类中方法调用，导致@Transactional失效

```
开发中避免不了会对同一个类里面的方法调用，比如有一个类Test，它的一个方法A，A再调用本类的方法B（不论方法B是用public还是private修饰），但方法A没有声明注解事务，而B方法有。则外部调用方法A之后，方法B的事务是不会起作用的。这也是经常犯错误的一个地方。

因为只有当事务方法被当前类以外的代码调用时，才会由`Spring`生成的代理对象来管理。
```

## 5.异常被你的 catch“吃了”导致@Transactional失效

```
如果B方法内部抛了异常，而A方法此时try catch了B方法的异常，那这个事务还能正常回滚吗？
```

## 6、数据库引擎不支持事务