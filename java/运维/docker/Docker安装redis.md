## Docker 安装redis

##### 1.搜索docker镜像(可以看到搜索的结果，这个结果是按照一定的星级评价规则排序的)

```
docker search redis
```

##### 2.拉取docker的mysql镜像(如果想指定版本号,需要到https://hub.docker.com/_/mysql?tab=tags查看版本号)

```
docker pull redis:5.0
```

##### 3.启动安装mysql

```
docker run -p 6379:6379 -v $PWD/data:/data  -d redis:5.0 redis-server --appendonly yes

-p 6379:6379 : 将容器的6379端口映射到主机的6379端口

-v $PWD/data:/data : 将主机中当前目录下的data挂载到容器的/data

redis-server --appendonly yes : 在容器执行redis-server启动命令，并打开redis持久化配置
```

##### 4.连接、查看容器,使用redis镜像执行redis-cli命令连接到刚启动的容器,主机IP为192.168.0.153

```
docker exec -it CONTAINERID(容器id) redis-cli
```

