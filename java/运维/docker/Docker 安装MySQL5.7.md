## Docker 安装MySQL5.7

##### 1.搜索docker镜像(可以看到搜索的结果，这个结果是按照一定的星级评价规则排序的)

```
docker search mysql
```

##### 2.拉取docker的mysql镜像(如果想指定版本号,需要到https://hub.docker.com/_/mysql?tab=tags查看版本号)

```
docker pull mysql:5.7.19
```

##### 3.启动安装mysql

```
sudo docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql:5.7

-p53306:3306：将容器的3306端口映射到主机的3306端口；

-e MYSQL_ROOT_PASSWORD=123456：初始化root用户的密码；

--name 给容器命名，mysql；

-d 表示容器在后台运行

注意事项:-d 后面是mysql镜像,一定要指定镜像版本号,也就是拉取的镜像版本

```

##### 4.如果还是不能连接，是局域网的问题，可以直接进入实例操作

```
grant all privileges on *.* to root@"%" identified by "123456" with grant option;
```

##### 5.进入容器

```
#进入容器
docker exec -it mysql bash
```

##### 6.登录mysql

```
#登录mysql
mysql -u root -p
```