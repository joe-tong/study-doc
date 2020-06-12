## Docker 安装 mongoDB

##### 1.搜索docker镜像(可以看到搜索的结果，这个结果是按照一定的星级评价规则排序的)

```
docker search mongo
```

##### 2.拉取docker的mongo镜像(如果想指定版本号,需要到https://hub.docker.com/_/mysql?tab=tags查看版本号)

```
docker pull mongo:4.0.9
```

##### 3.启动安装mongoDB

```
docker run -p 27017:27017 -v $PWD/db:/data/db -d mongo:4.0.9

命令说明：

-p 27017:27017 :将容器的27017 端口映射到主机的27017 端口

-v $PWD/db:/data/db :将主机中当前目录下的db挂载到容器的/data/db，作为mongo数据存储目录
```

##### 4.连接、查看容器,使用命令连接到刚启动的容器,主机IP为192.168.0.153

```
docker run -it mongo:4.0.9 mongo --host 192.168.0.153
```

