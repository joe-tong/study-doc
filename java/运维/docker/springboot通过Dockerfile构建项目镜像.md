## springboot通过Dockerfile构建项目镜像

1.idea下载docker插件

2.docker配置连接地址（测试是否能连接到docker）

```
如果连接不到需要改docker.service配置暴露端口
ExecStart=/usr/bin/dockerd -H unix://var/run/docker.sock -H tcp://0.0.0.0:2375
```

3.在项目的根目录下创建Dockerfile文件



