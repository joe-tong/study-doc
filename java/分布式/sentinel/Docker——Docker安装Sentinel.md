# Docker——Docker安装Sentinel

Sentinel是面向分布式服务框架的轻量级流量控制框架,主要以流量为切入点,从流量控制,熔断降级,系统负载保护等多个维度来维护系统的稳定性.

1. 拉取镜像：

```
docker pull bladex/sentinel-dashboard
```

2. 运行镜像：

```
docker run --name sentinel -d -p 8858:8858 -d bladex/sentinel-dashboard
```

3. 访问dashboard 地址：http://localhost:8858
   账号密码都为：sentinel
   ![Sentinel页面](https://img-blog.csdnimg.cn/2020010920412552.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgzNTY1OQ==,size_16,color_FFFFFF,t_70)