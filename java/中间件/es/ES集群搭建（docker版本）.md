# ES集群搭建（docker版本）

## 1.拉去ES镜像(服务A、服务B)

```
docker pull elasticsearch:6.7.1
```

## 2.创建目录(服务A、服务B)

```
mkdir -p /ES/config #挂载配置文件
mkdir -p /ES/plugins #挂载存放分词插件
mkdir -p /ES/data #挂载数据

chmod -R 777 /ES
```

## 3.配置es.yml(服务A、服务B)

````
cd /ES/config
vim es.yml
```

es.yml (服务A)

```
cluster.name: elasticsearch-cluster
node.name: es-node1
network.bind_host: 0.0.0.0
network.publish_host: 192.168.91.66
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true 
node.data: true  
discovery.zen.ping.unicast.hosts: ["192.168.91.66:9300","192.168.91.66:930"]
discovery.zen.minimum_master_nodes: 1
```

es.yml(服务B)

```
cluster.name: elasticsearch-cluster
node.name: es-node1
network.bind_host: 0.0.0.0
network.publish_host: 192.168.91.66
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true 
node.data: true  
discovery.zen.ping.unicast.hosts: ["192.168.91.66:9300","192.168.91.66:9300"]
discovery.zen.minimum_master_nodes: 1
```

## 4.elasticsearch用户拥有的内存权限太小，至少需要262144

在/etc/sysctl.conf文件最后添加一行

```
vm.max_map_count=262144
```

## 4.启动docker(服务A、服务B)

```
docker run -e -d -p 9200:9200 -p 9300:9300 -v  /ES/config/es.yml:/usr/share/elasticsearch/config/elasticsearch.yml  -v /ES/plugins:/usr/share/elasticsearch/plugins -v /ES/data:/usr/share/elasticsearch/data --name ES01 e2667f5db289
```

判断是否集群搭建成功： http://192.168.0.182:9200/_cat/nodes 

## 5.安装Ik中文分词器

 https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v6.7.1  下载

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596097573406.png)

```
解压到挂载插件目录
cd /ES/plugins/
mkdir ik
cd ik/
unzip elasticsearch-analysis-ik-6.7.1.zip
```

重启es

```
#在kibana的devTools的console输入：

POST _analyze
{
"analyzer":"ik_max_word",
"text": "中华人民共和国人民大会堂"
}

分词了说明成功了

```



## 6.拉去kibana

```
docker pull kibana:6.7.1
```

6.运行kibana

```
docker run --name tlkiba -e ELASTICSEARCH_HOSTS=http://当前服务器ip:9200 -e SERVER_PORT=5601  -e SERVER_HOST=0.0.0.0 -p 5601:5601 -d 7f92ab934206
```

 访问地址： [http://192.168.0.182:5601](http://192.168.0.182:5601/) 

## 7.安装head插件

```
docker pull mobz/elasticsearch-head:5

docker run -d -p 9100:9100 docker.io/mobz/elasticsearch-head:5
```

 访问地址：http://192.168.0.182:9100