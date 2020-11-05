# seata server 部署（docker版本）

1、拉取镜像

```
docker pull seataio/seata-server:1.3.0
```

2、运行镜像

```
docker run --name seata1.3.0 -p 8091:8091 -d  seataio/seata-server:1.3.0
```

3、复制配置文件到主机

```
docker cp seata1.3.0:/seata-server  /seata/docker/seata-server
```

4、停止服务

```
docker stop seata1.3.0
```

5、删除服务

```
docker rm seata1.3.0
```

6、重新运行服务，至此服务已经启动完成，接下来就是在/seata/docker/seata-server目录中修改对应的配置（设置开机自启和关键配置挂载到本地目录方便修改配置）

```
docker run -d --restart always  --name  seata1.3.0 -p 8091:8091  -v /seata/docker/seata-server:/seata-server -e SEATA_IP=192.168.66.5 -e SEATA_PORT=8091 --privileged=true seataio/seata-server:1.3.0
```

7、切换到/seata/docker/seata-server/resources配置目录

```
cd /seata/docker/seata-server/resources
```

7、修改registry.conf，配置注册中心实现HA（这里改成nacos）

```
vim registry.conf
```

注册中心-修改部分：

```
registry {
  type = "nacos"
  nacos {
    application = "seata-server"
    serverAddr = "192.168.66.5:8848"
    group = "SEATA_GROUP"
    cluster = "default"
  }
}
```

8、修改registry.conf和file.config, 配置中心（这里改成db）

```
vim registry.conf
```

指定配置:

```
config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "file"
  }
```

修改file.config：

```
store {
  ## store mode: file、db、redis
  mode = "db"
  
  db {
    ## the implement of javax.sql.DataSource, such as         DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://192.168.66.5:3306/seata"
    user = "root"
    password = "123456"
    minConn = 5
    maxConn = 30
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }
 }
```

9、重启服务

```
docker restart seata-server 重启
docker logs seata-server #查看启动日志
```









