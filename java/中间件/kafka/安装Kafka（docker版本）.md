# 安装Kafka（docker版本）

#### 下载zookeeper镜像

```
docker pull wurstmeister/zookeeper
```

#### 启动镜像生成容器

```
docker run -d --name zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime wurstmeister/zookeeper
```

#### 下载kafka镜像

```
docker pull wurstmeister/kafka
```

#### 启动镜像生成容器

```
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.66.5:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.66.5:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -v /etc/localtime:/etc/localtime wurstmeister/kafka
```

```
参数说明：
-e KAFKA_BROKER_ID=0  在kafka集群中，每个kafka都有一个BROKER_ID来区分自己

-e KAFKA_ZOOKEEPER_CONNECT=172.16.0.13:2181/kafka 配置zookeeper管理kafka的路径172.16.0.13:2181/kafka

-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.16.0.13:9092  把kafka的地址端口注册给zookeeper，如果是远程访问要改成外网IP,类如Java程序访问出现无法连接。

-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口

-v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间
```

#### 验证kafka是否可以使用

```
验证kafka是否可以使用

进入容器
docker exec -it kafka bash

进入 /opt/kafka_2.12-2.3.0/bin/ 目录下
cd /opt/kafka_2.12-2.3.0/bin/

运行kafka生产者发送消息
./kafka-console-producer.sh --broker-list localhost:9092 --topic sun

发送消息
{"datas":[{"channel":"","metric":"temperature","producer":"ijinus","sn":"IJA0101-00002245","time":"1543207156000","value":"80"}],"ver":"1.0"}
 
运行kafka消费者接收消息
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic sun --from-beginning
```

熟悉junit单元测试等主流测试框架、熟悉网络通讯技术Socket、Http、了解NIO、Netty、WebSocket