# MQ系列-Kafka篇章

## 1.Kafka基本概念

```
kafka是一个分布式的，分区的消息(官方称之为commit log)服务。它提供一个消息系统应该具备的功能，但是确有着独 特的设计。可以这样来说，Kafka借鉴了JMS规范的思想，但是确并没有完全遵循JMS规范
```

首先，让我们来看一下基础的**消息(Message)相关术语**： 

## 2.消息(Message)相关术语

- **解释**Broker 

```
消息中间件处理节点，一个Kafka节点就是 一个broker，一个或者多个Broker可以组 成一个Kafka集群 
```

- Topic 

```
Kafka根据topic对消息进行归类，发布到 Kafka集群的每条消息都需要指定一个topic 
```

- Producer 

```
消息生产者，向Broker发送消息的客户端 
```

- Consumer 

```
消息消费者，从Broker读取消息的客户端 
```

- ConsumerGroup 

```
每个Consumer属于一个特定的Consumer Group，一条消息可以被多个不同的 Consumer Group消费，但是一个 Consumer Group中只能有一个Consumer 能够消费该消息 
```

- Partition 

```
物理上的概念，一个topic可以分为多个 partition，每个partition内部消息是有序
```

因此，从一个较高的层面上来看，producer通过网络发送消息到Kafka集群，然后consumer来进行消费，如下图：

![kafka.png](https://ae03.alicdn.com/kf/Uc360e6b25bc3465aad4d7bfb8634b341R.jpg)

服务端(brokers)和客户端(producer、consumer)之间通信通过**TCP协议**来完成。 

## 3.主题Topic和消息日志Log

​      让我们首先深入理解Kafka提出一个高层次的抽象概念-Topic。 

可以理解**Topic是一个类别的名称**，同类消息发送到同一个Topic下面。对于每一个Topic，下面可以有多个分区 

## 4.Producers

​       生产者将消息发送到topic中去，同时负责选择将message发送到topic的哪一个partition中。通过round­robin做简单的 负载均衡。也可以根据消息中的某一个关键字来进行区分。通常第二种方式使用的更多

## 5.Consumers 

传统的消息传递模式有2种：队列( queue) 和（publish-subscribe） 

- queue模式：多个consumer从服务器中读取数据，消息只会到达一个consumer。 

- publish-subscribe模式：消息会被广播给所有的consumer。

![copy.png](https://ae04.alicdn.com/kf/U072fde201f85472aa57377dfa817dbdcg.jpg)

上图说明：由2个broker组成的kafka集群，总共有4个partition(P0-P3)。这个集群由2个Consumer Group， A有2个 

consumer instances ，B有四个。 

通常一个topic会有几个consumer group，每个consumer group都是一个逻辑上的订阅者（ logical 

subscriber ）。每个consumer group由多个consumer instance组成，从而达到可扩展和容灾的功能。

### 5.1消费顺序

Kafka比传统的消息系统有着更强的**顺序保证**。 

**一个partition同一个时刻在一个consumer group中只有一个consumer instance在消费，从而保证顺序**。 

**consumer group中的consumer instance的数量不能比一个Topic中的partition的数量多，否则，多出来的** 

**consumer消费不到消息**。 

Kafka只在partition的范围内保证消息消费的局部顺序性，不能在同一个topic中的多个partition中保证总的消费顺序 

性。

如果有在总体上保证消费顺序的需求，那么我们可以通过将topic的partition数量设置为1，将consumer group中的 

consumer instance数量也设置为1。 



## 6.实战

### 6.1 添加依赖

```
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
```

### 6.2 application.yml

```
spring:  
  kafka:
    # 以逗号分隔的地址列表，用于建立与Kafka集群的初始连接(kafka 默认的端口号为9092)
    bootstrap-servers: 192.168.153:9092
    producer:
      # 发生错误后，消息重发的次数。
      retries: 0
      #当有多个消息需要被发送到同一个分区时，生产者会把它们放在同一个批次里。该参数指定了一个批次可以使用的内存大小，按照字节数计算。
      batch-size: 16384
      # 设置生产者内存缓冲区的大小。
      buffer-memory: 33554432
      # 键的序列化方式
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      # 值的序列化方式
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      # acks=0 ： 生产者在成功写入消息之前不会等待任何来自服务器的响应。
      # acks=1 ： 只要集群的首领节点收到消息，生产者就会收到一个来自服务器成功响应。
      # acks=all ：只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。
      acks: 1
    consumer:
      # 自动提交的时间间隔 在spring boot 2.X 版本中这里采用的是值的类型为Duration 需要符合特定的格式，如1S,1M,2H,5D
      auto-commit-interval: 1S
      # 该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该作何处理：
      # latest（默认值）在偏移量无效的情况下，消费者将从最新的记录开始读取数据（在消费者启动之后生成的记录）
      # earliest ：在偏移量无效的情况下，消费者将从起始位置读取分区的记录
      auto-offset-reset: earliest
      # 是否自动提交偏移量，默认值是true,为了避免出现重复数据和数据丢失，可以把它设置为false,然后手动提交偏移量
      enable-auto-commit: false
      # 键的反序列化方式
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      # 值的反序列化方式
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    listener:
      # 在侦听器容器中运行的线程数。
      concurrency: 5
      ack-mode: manual_immediate
      
shop-mall:
  kafka:
    topic:
      group-id: topicGroupId
      topic-name:
        - topic1
        - topic2
        - topic3      
```

### 6.3 KafkaTopicConfiguration和KafkaTopicProperties

```
@Configuration
@EnableConfigurationProperties(KafkaTopicProperties.class)
public class KafkaTopicConfiguration {

    private final KafkaTopicProperties properties;

    public KafkaTopicConfiguration(KafkaTopicProperties properties) {
        this.properties = properties;
    }

    @Bean
    public String[] kafkaTopicName() {
        return properties.getTopicName();
    }

    @Bean
    public String topicGroupId() {
        return properties.getGroupId();
    }

}
```

```
@ConfigurationProperties("shop-mall.kafka.topic")
public class KafkaTopicProperties implements Serializable {

    private String groupId;
    private String[] topicName;

    public String getGroupId() {
        return groupId;
    }

    public void setGroupId(String groupId) {
        this.groupId = groupId;
    }

    public String[] getTopicName() {
        return topicName;
    }

    public void setTopicName(String[] topicName) {
        this.topicName = topicName;
    }
}
```

### 6.4 KafkaService

```
@Service
@Slf4j
public class KafkaService {


    private  KafkaTemplate<Integer, String> kafkaTemplate;

    /**
     * 注入KafkaTemplate
     * @param kafkaTemplate kafka模版类
     */
    @Autowired
    public void IndicatorService(KafkaTemplate kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }


    @KafkaListener(topics = "#{kafkaTopicName}", groupId = "#{topicGroupId}")
    public void processMessage(ConsumerRecord<Integer, String> record) {
        log.info("kafka processMessage start");
        log.info("processMessage, topic = {}, msg = {}", record.topic(), record.value());

        // do something ...

        log.info("kafka processMessage end");
    }

    public void sendMessage(String topic, String data) {
        log.info("kafka sendMessage start");
        ListenableFuture<SendResult<Integer, String>> future = kafkaTemplate.send(topic, data);
        future.addCallback(new ListenableFutureCallback<SendResult<Integer, String>>() {
            @Override
            public void onFailure(Throwable ex) {
                log.error("kafka sendMessage error, ex = {}, topic = {}, data = {}", ex, topic, data);
            }

            @Override
            public void onSuccess(SendResult<Integer, String> result) {
                log.info("kafka sendMessage success topic = {}, data = {}",topic, data);
            }
        });
        log.info("kafka sendMessage end");
    }
}
```