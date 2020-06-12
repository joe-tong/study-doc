## **Docker Compose****介绍** 

使用微服务架构的应用系统一般包含若干个微服务，每个微服务一般都会部署多个实例。如果每个微服务都要手动启 停，那么效率之低、维护量之大可想而知。本节课将讨论如何使用 Docker Compose来轻松、高效地管理容器。为了 简单起见将 Docker Compose简称为 Compose。 Compose 是一个用于定义和运行多容器的Docker应用的工具。使用Compose，你可以在一个配置文件（yaml格式） 中配置你应用的服务，然后使用一个命令，即可创建并启动配置中引用的所有服务。下面我们进入Compose的实战吧 

### **Docker Compose****的安装** 

Compose的安装有多种方式，例如通过shell安装、通过pip安装、以及将compose作为容器安装等等。本文讲解通过 shell安装的方式。其他安装方式如有兴趣，可以查看Docker的官方文档： https://docs.docker.com/compose/install/ 

```
1 # docker compose安装步骤 

2 sudo curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose 

3 sudo chmod +x /usr/local/bin/docker-compose

4 docker‐compose ‐‐version 
```

### **Docker Compose****入门示例** 

Compose的使用非常简单，只需要编写一个docker-compose.yml，然后使用docker-compose 命令操作即可。docker- compose.yml描述了容器的配置，而docker-compose 命令描述了对容器的操作。我们首先通过一个示例快速入门： 还记得上节课，我们使用Dockerfile为项目microservice-eureka-server构建Docker镜像吗？我们还以此项目为例测试 

我们在microservice-eureka-server-0.0.1-SNAPSHOT.jar所在目录的上一级目录，创建docker-compose.yml 文 

件。 

目录树结构如下： 

├── docker-compose.yml 

└── eureka 

├── Dockerfile 

└── microservice-eureka-server-0.0.1-SNAPSHOT.jar 

- 然后在docker-compose.yml 中添加内容如下： 

```
version: '3' 
 services: 
  eureka: #指定服务名 
  image: microservice‐eureka‐server:0.0.1 #指定镜像名称 
  build: ./eureka #指定Dockfile所在路径 
  ports: 
   ‐ "8761:8761" #指定端口映射 
  expose: 
   ‐ 8761 #声明容器对外暴露的端口 
```

- 在docker-compose.yml 所在路径执行： 

```
docker‐compose up （后面加‐d可以后台启动)
```

