## Docker常用命令

#### 查看Docker所有正在运行的容器

```
docker ps
```

#### 查看Docker已退出的容器

```
docker ps -a
```

#### 查看Docker所有镜像

```
docker images
```

#### 删除镜像

删除镜像之前一定要删除容器

```
1.docker ps -a 查看容器
2.docker rm CONTAINERID 删除容器
3.docker images 查看镜像
4.docker rmi IMAGEID 删除镜像
```

#### 停止容器

```
docker stop CONTAINERID
```

