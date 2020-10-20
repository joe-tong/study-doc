# docker部署apollo详细教程(亲测)

 **作者：星晴（当地小有名气，小到只有自己知道的杰伦粉）**

## 1、前言

apollo的详细介绍我就不在这里多说了，官网上[https://github.com/ctripcorp/...](https://github.com/ctripcorp/apollo)已经说的非常明白了，我就不在这班门弄斧了，还不了解的小伙伴可以去官网上去了解下。

本篇文章只是记录我在使用docker部署的Apollo以及其集群的方式，给大家分享出来也自我做一个记录。

注意： 我是直接部署开始的，有关数据库的创建和初始化自己根据官网搞定。

## 2、源码编译

### 2.1 下载源码

github：https://github.com/ctripcorp/apollo

### 2.2 修改源码配置

#### 2.2.1 网络策略

网络策略直接使用官网描述的就可以，具体就是分别编辑apollo-configservice/src/main/resources/application.yml和apollo-adminservice/src/main/resources/application.yml，然后把需要忽略的网卡加进去。

如下面这个例子就是对于apollo-configservice，把docker0和veth.*的网卡在注册到Eureka时忽略掉。

```
spring:
      application:
          name: apollo-configservice
      profiles:
        active: ${apollo_profile}
      cloud:
        inetutils:
          ignoredInterfaces:
            - docker0
            - veth.*
```

> 注意，对于application.yml修改时要小心，千万不要把其它信息改错了，如spring.application.name等。

### 2.2.2 动态指定注册网络

在使用docker搭建集群是， adminservice、configservice都需要向注册中心注册地址，如果不指定注册IP，注册的是docker内部的网络，导致网络不通。
在apollo-configservice/src/main/resources/bootstrap.yml和apollo-adminservice/src/main/resources/bootstrap.yml添加如下代码。

```
eureka:
  instance:
        ip-address: ${eureka.instance.ip-address}
```

这个地方取值从环境变量中取，容器外部来配置这样给部署带来了更大的灵活性。

到这源码的修改已经完成，直接build打包就可以了，拿到对应三个服务的zip包。

## 3.创建数据库

#### 3.1 ApolloConfigDB

源码下的sql文件路径，直接执行：

```
apollo-resourse/apollo/scripts/docker-quick-start/sql/apolloconfigdb.sql
```

配置ServerConfig的eureka：

```
INSERT INTO `ApolloConfigDB`.`ServerConfig`(`Id`, `Key`, `Cluster`, `Value`, `Comment`, `IsDeleted`, `DataChange_CreatedBy`, `DataChange_CreatedTime`, `DataChange_LastModifiedBy`, `DataChange_LastTime`) VALUES (1, 'eureka.service.url', 'default', 'http://localhost:8080/eureka/', 'Eureka服务Url，多个service以英文逗号分隔', b'0', 'default', '2020-09-23 09:44:28', '', '2020-10-16 06:58:59');
```

#### 3.2 ApolloPortalDB

源码下的sql文件路径，直接执行：

```
apollo/scripts/docker-quick-start/sql/apolloportaldb.sql
```

配置ServerConfig的env：

```
INSERT INTO `ApolloPortalDB`.`ServerConfig`(`Id`, `Key`, `Value`, `Comment`, `IsDeleted`, `DataChange_CreatedBy`, `DataChange_CreatedTime`, `DataChange_LastModifiedBy`, `DataChange_LastTime`) VALUES (1, 'apollo.portal.envs', 'dev,fat,uat,lpt,pro', '可支持的环境列表', b'0', 'default', '2020-09-23 09:40:28', '', '2020-10-16 06:28:03');
```

## 4、dockerfile编写

Apollo 的Dockerfile非常简单， 直接使用官方提供的即可。下方是adminservice示例。

```
# Dockerfile for apollo-adminservice
# Build with:
# docker build -t apollo-adminservice .
# Run with:
# docker run -p 8090:8090 -d --name apollo-adminservice apollo-adminservice

FROM java:8-jre
MAINTAINER Louis

ENV VERSION 1.5.0

RUN apt-get install unzip

ADD apollo-adminservice-${VERSION}-github.zip /apollo-adminservice/apollo-adminservice-${VERSION}-github.zip

RUN unzip /apollo-adminservice/apollo-adminservice-${VERSION}-github.zip -d /apollo-adminservice \
    && rm -rf /apollo-adminservice/apollo-adminservice-${VERSION}-github.zip \
    && sed -i '$d' /apollo-adminservice/scripts/startup.sh \
    && echo "tail -f /dev/null" >> /apollo-adminservice/scripts/startup.sh

EXPOSE 8090

CMD ["/apollo-adminservice/scripts/startup.sh"]
```

> 需要注意的，
> 1: version 需要根据自己打包的版本来修改 (源码的pom里面有版本号)
> 2: ADD zip包时修改路径【ADD   本地文件的路径  docker容器内的路径】  

## 5 docker-compose 的编写

### 5.1 apollo-configservice-compose.yml

```
version: "3"
services:
  apollo-configservice:
    container_name: apollo-configservice
    build: apollo-configservice/
    image: apollo-configservice
    ports:
      - 8080:8080
    volumes:
      - "/docker/apollo/logs/100003171:/opt/logs/100003171"
    environment:
      - spring_datasource_url=jdbc:mysql://127.0.0.1:8306/ApolloConfigDB_TEST?characterEncoding=utf8
      - spring_datasource_username=root
      - spring_datasource_password=mysql2019*
      - eureka.instance.ip-address=172.11.11.11

    restart: always
```

> 注意事项，
> 1: build: 中指定你Dockerfile文件的位置
> 2: environment 环境变量中指定你数据库的配置信息
> 3: eureka.instance.ip-address 指定注册到eureka地址，这个最好使用你物理机的内网地址。

> 特别注意： 启动前最好先修改ApolloConfigDB数据库中 ServerConfig中的eureka.service.url值，改为具体的IP

启动：

```
docker-compose -f apollo-configservice-compose.yml up --build -d
```

### 5.2 apollo-adminservice-compose.yml

apollo-adminservice-compose.yml的内容基本和apollo-configservice-compose.yml相同，在这我就不一一说明了。

### 5.3 apollo-portal-compose.yml

```
version: "3"
services:
  apollo-portal:
    container_name: apollo-portal
    build: apollo-portal/
    image: apollo-portal
    ports:
      - 8070:8070
    volumes:
      - "/docker/apollo/logs/100003173:/opt/logs/100003173"
      - "/apollo-portal/config/apollo-env.properties:/apollo-portal/config/apollo-env.properties"
    environment:
      - spring_datasource_url=jdbc:mysql://127.0.0.1:8306/ApolloPortalDB?characterEncoding=utf8
      - spring_datasource_username=root
      - spring_datasource_password=mysql2019*
      

    restart: always
```

> 注意事项：
> 1: 需要注意的和上述configservice基本相同
> 2: 特别需要注意的事项 重要！重要！重要！重要！重要！volumes： 中我将
> apollo-env.properties文件映射到容器外面了，将自己的apollo-env.properties文件配置后将自己的挂载地址填上，冒号前的地址“/apollo-portal/config/apollo-env.properties”修改成自己的。必须在启动前将此配置文件指定好。

启动

```
docker-compose -f apollo-configservice-compose.yml up --build -d
```

#### 5.3.1 apollo-env.properties

别忘记修改挂载/apollo-portal/config/apollo-env.properties,把${dev_meta} = http://localhost:8080 根据自己服务进行修改

```
local.meta=http://localhost:8080
dev.meta=${dev_meta}
fat.meta=${fat_meta}
uat.meta=${uat_meta}
lpt.meta=${lpt_meta}
pro.meta=${pro_meta}
```

将自己的meta地址配置上， 没有的可以直接删除。有不明白的可以去官网上了解，环境配置完后修改对应的数据库中ApolloPortalDB.ServerConfig
中apollo.portal.envs 值，填上你的配置的环境。否则我们在portal管理页面只能看到默认dev环境。

### 5.4 完整的docker-compose.yml

如果嫌弃一个个启动麻烦也以使用一个完整的compose来启动。

```
version: "3"
services:
  apollo-configservice:
    container_name: apollo-configservice
    build: apollo-configservice/
    image: apollo-configservice
    ports:
      - 8080:8080
    volumes:
      - "/docker/apollo/logs/100003171:/opt/logs/100003171"
    environment:
      - spring_datasource_url=jdbc:mysql://47.xx.xx.209:8306/ApolloConfigDB?characterEncoding=utf8
      - spring_datasource_username=root
      - spring_datasource_password=Tusdao@xx*
      - eureka.instance.ip-address=172.11.11.11
    restart: always

  apollo-adminservice:
    container_name: apollo-adminservice
    build: apollo-adminservice/
    image: apollo-adminservice
    ports:
      - 8090:8090
    volumes:
      - "/docker/apollo/logs/100003172:/opt/logs/100003172"
    environment:
      - spring_datasource_url=jdbc:mysql://47.xx.xx.209:8306/ApolloConfigDB?characterEncoding=utf8
      - spring_datasource_username=root
      - spring_datasource_password=Tusdao@xx*
      - eureka.instance.ip-address=172.11.11.11
    depends_on:
      - apollo-configservice

    restart: always

  apollo-portal:
    container_name: apollo-portal
    build: apollo-portal/
    image: apollo-portal
    ports:
      - 8070:8070
    volumes:
      - "/docker/apollo/logs/100003173:/opt/logs/100003173"
      - "/Apollo/docker-image/apollo-portal/config/apollo-env.properties:/apollo-portal/config/apollo-env.properties"
    environment:
      - spring_datasource_url=jdbc:mysql://47.xx.xx.209:8306/ApolloPortalDB?characterEncoding=utf8
      - spring_datasource_username=root
      - spring_datasource_password=Tusdao@xx*
    depends_on:
      - apollo-adminservice
    restart: always
```

> 注意： 需要修改的地方和单个基本相同，我在这就不唠叨了。

到这docker部署Apoll基本搞定，如有小伙伴需要完整的docker部署文件请移步[https://github.com/yuelicn/do...](https://github.com/yuelicn/docker-apollo)

## 6.集群的搭建

Apollo集群的搭建非常简单，只需要修改两个地方就可以了，我们就以正式环境(pro)来说明，
在pro环境我们搭建了两套adminservice、configservice,数据库都是同一个ApolloConfigDB，

1:将ServerConfig中的eureka.service.url值eureka连接信息两个都写上用逗号分隔：[http://IP-1](http://ip-1/):port/eureka,[http://IP-2](http://ip-2/):port/eureka

2:修改apollo-env.properties中对应环境的连接信息如： pro.meta=[http://IP-1](http://ip-1/):port,[http://IP-2](http://ip-2/):port 地址用逗号分隔就可以了。

之后重启服务就搞定了。

最后强调，adminservice、configservice 需要每个环境单独部署，包括数据库。portal只需要部署一套就可以了。

OK! 完成，上述是指个人搭建记录，希望对你有帮助，如果不对的地方欢迎指正。

