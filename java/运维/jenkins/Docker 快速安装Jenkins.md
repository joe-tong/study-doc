# Docker 快速安装Jenkins

**一、前言**

　　有人问，为什么要用Jenkins？我说下我以前开发的痛点，在一些中小型企业，每次开发一个项目完成后，需要打包部署，可能没有专门的运维人员，只能开发人员去把项目打成一个war包，可能这个项目已经上线了，需要把服务关，在部署到服务器上，将项目启动起来，这个时候可能某个用户正在操作某些功能上的东西，如果你隔三差五的部署一下，这样的话对用户的体验也不好，自己也是烦的很，总是打包拖到服务器上。希望小型企业工作人员学习一下，配置可能复杂，但是你配置好了之后，你只需要把代码提交到Git或者Svn上，自动构建部署，非常方便。有任何地方不懂的翻到最下方随时咨询我，想帮助更多的初学者共同一起努力成长！

**二、Jenkins简介**

　　Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。


**三、jenkins基本工作原理**

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml7784\wps1.jpg) 

 

 以上为基本工作原理，只是为了开发人员更好的理解画的（个人理解），详情可查看官方文档：<https://www.jenkins.io/zh/>

**四、准备工作**

1.需要准备一台服务器，大家可以在网上买，个人学习的话还是建议大家去安装一个虚拟机，去装一个Linux系统。

2.需要准备一个远程连接工具，连接到Linux系统，作者采用的是：Xshell 工具 （注意：Xshell下载地址翻到本文最下面）。

3.安装好docker

**五、开始安装**

1.启动docker，下载Jenkins镜像文件

```
docker pull jenkins/jenkins
```

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml7784\wps2.jpg) 

 

 

 2.创建Jenkins挂载目录并授权权限（我们在服务器上先创建一个jenkins工作目录 /var/jenkins_mount，赋予相应权限，稍后我们将jenkins容器目录挂载到这个目录上，这样我们就可以很方便地对容器内的配置文件进行修改。 如果我们不这样做，那么如果需要修改容器配置文件，将会有点麻烦，因为虽然我们可以使用docker exec -it --user root 容器id /bin/bash 命令进入容器目录，但是连简单的 vi命令都不能使用）

```
mkdir -p /var/jenkins_home

chmod 777 /var/jenkins_home
```

3.创建并启动Jenkins容器

　　**-d 后台运行镜像**

　　**-p 10240:8080 将镜像的8080端口映射到服务器的10240端口。**

　　**-p 10241:50000 将镜像的50000端口映射到服务器的10241端口**

　　**-v /var/jenkins_home:/var/jenkins_home目录为容器jenkins工作目录，我们将硬盘上的一个目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。这里我们设置的就是上面我们创建的 /var/jenkins_mount目录**

　　**-v /etc/localtime:/etc/localtime让容器使用和服务器同样的时间设置。**

　　**--name myjenkins 给容器起一个别名**

​        **--privileged=true**  **jenkins容器有内部执行权限**

```CQL
docker run -d -p 10240:8080 -p 10241:50000 -v /var/jenkins_home:/var/jenkins_home -v /etc/localtime:/etc/localtime /var/run/docker.sock:/var/run/docker.sock --privileged=true --name myjenkins jenkins/jenkins
```

把maven放到挂载目录，以便docker能执行mvn

```
cp /usr/local/apache-maven-3.6.3 /var/jenkins_home/
```



![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml7784\wps3.jpg) 

 4.查看jenkins是否启动成功，如下图出现端口号，就为启动成功了

```
docker ps -l
```

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml7784\wps4.jpg) 

 5.查看docker容器日志。

```
docker logs myjenkins
```

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml7784\wps5.jpg) 

 6.配置镜像加速，进入 cd /var/jenkins_home/ 目录。

```
cd /var/jenkins_home/
```

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml7784\wps6.jpg) 

**修改里的内容**

```
vi  hudson.model.UpdateCenter.xml
```

**修改前**

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml7784\wps7.jpg) 

将 url 修改为 清华大学官方镜像：https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json    或者 http://mirror.esuni.jp/jenkins/updates/update-center.json

**修改后**

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml7784\wps8.jpg) 

 

 7.访问Jenkins页面，输入你的ip加上10240

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml7784\wps9.jpg) 

 

 8.管理员密码获取方法，编辑initialAdminPassword文件查看，把密码输入登录中的密码即可，开始使用。

```
vi /var/jenkins_home/secrets/initialAdminPassword
```







