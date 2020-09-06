# Dubbo

# 1.负载均衡

随机+权重

最小使用数

一致性哈希算法

# 2.异步调用

1.提高服务调用性能

2.服务降级

# 3.dubbo协议

#### 3.1 Dubbo是什么类型的协议？

```
二进制协议

优势：体积小，没有请求头，解析更快,传输更快
```

#### 3.2 dubbo都支持什么协议，推荐用哪种？

- dubbo://（推荐）
- rmi://
- hessian://
- http://
- webservice://
- thrift://
- memcached://
- redis://
- rest://

# 4.远程传输

Netty实现的

# 5.Dubbo 节点

#### 5.1 Dubbo里面有哪几种节点角色？

```
provider 
consumer
registry
monitor
container
```

#### 5.2 在provider上可以配置的consumer端的属性有哪些？

- timeout: 方法调用超时
- retiries : 重试次数
- loadbalance : 负载均衡算法
- actives: 消费端最大并发调用次数



#### 5.3 Dubbo推荐使用什么序列化框架，你知道的还有哪些？

```
推荐使用Hession序列化，还有Dubbo、FastJson、Java自带序列化
```

#### 5.4 Dubbo默认使用的是什么通信框架，还有别的选择吗？

```
Dubbo 默认使用 Netty 框架，也是推荐的选择，另外内容还集成有Mina、Grizzly。
```

### 5.5 当一个服务接口有多种实现时怎么做？

```
当一个接口有多种实现时，可以用 group 属性来分组，服务提供方和消费方都指定同一个 group 即可。
```



