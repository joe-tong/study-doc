# Nacos的部署（docker版本）

## 1.拉取nacos

```
git clone https://github.com/nacos-group/nacos-docker.git 
```

## 2.docker-compose 启动

```
cd nacos-docker
docker-compose -f example/standalone-mysql.yaml up
```

## 3.访问nacos

```
http://ip:8848/nacos  
username and password  都是nacos
```