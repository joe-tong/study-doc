# Linux安装maven

## 一、简介

　　 Maven是意第绪语，意思是“知识的积累者”，最初是为了简化Jakarta Turbine项目中的构建过程。有几个项目，每个项目都有自己的Ant构建文件，所有项目都略有不同。JAR已检入CVS。我们想要一种标准的方式来构建项目，清晰地定义项目的组成，一种简单的方式来发布项目信息，以及一种在多个项目中共享JAR的方式。

　　结果是一个可以用于构建和管理任何基于Java的项目的工具。我们希望我们已经创建了一些东西，可以使Java开发人员的日常工作更加轻松，并且通常有助于理解任何基于Java的项目。

## 二、准备工作

1.maven官方下载地址如下：（注意：maven下载地址翻到本文最下面）

https://maven.apache.org/download.cgi

 ![](https://ftp.bmp.ovh/imgs/2020/08/ce72a3b7a6b83bcb.jpg)  

 

##  三、开始安装

1.将下载好的maven安装包放在磁盘的 /usr/local/ 目录下，如下图：

 ![](https://ftp.bmp.ovh/imgs/2020/08/330c7095451453f9.jpg)  

2.解压apache-maven-3.6.3-bin.tar.gz文件。如下图：

```
tar -zxvf apache-maven-3.6.3-bin.tar.gz

```

![wps13.jpg](https://ae04.alicdn.com/kf/Ucbf9ac46417f4493988dda3eeb5552c29.jpg) 

3.配置maven仓库，设置阿里镜像仓库，一定要配置一下，国内的下载jar快些，首先进入cd apache-maven-3.6.3目录，创建仓库存储目录，mkdir ck。如下图：

```
cd apache-maven-3.6.3   #进入apache-maven-3.6.3目录
```

```
mkdir ck    #创建ck目录 
```

![wps14.jpg](https://ae02.alicdn.com/kf/Uc8840b954a9740cea91ca32cd510a8a7I.jpg)

4.进入cd conf目录，编辑 vi settings.xml文件，找到·localRepository下面复制一行加上<localRepository>/usr/local/apache-maven-3.6.3/ck</localRepository>， 在找到mirror 加上阿里的仓库配置，配置完成报错退出，如下图：

```
cd conf            # 进入conf目录

vi settings.xml # settings.xm文件

<localRepository>/usr/local/apache-maven-3.6.3/ck</localRepository>

<mirror>
  <id>alimaven</id>
  <name>aliyun maven</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
  <mirrorOf>central</mirrorOf>
</mirror> 
```

![wps16.jpg](https://ae03.alicdn.com/kf/U7d8a7ba35f1b4c55b5627089dd5588a1M.jpg) 

5.配置maven环境变量，编辑：vi /etc/profile 文件，翻到最后一行加上 export MAVEN_HOME=/usr/local/apache-maven-3.6.3  export PATH=$PATH:$MAVEN_HOME/bin  保存退出，如下图：

```
vi /etc/profile
```

```
export MAVEN_HOME=/usr/local/apache-maven-3.6.3

export PATH=$PATH:$MAVEN_HOME/bin 
```

![wps17.jpg](https://ae04.alicdn.com/kf/Ufe0056d404db40e791c7ff16d064a0bfS.jpg)

6.重新加载一下，source /etc/profile 使新增配置生效，如下：

```
source /etc/profile
```



7.到此以安装完成，测试一下，输入命令：mvn -v ，如下：

```
mvn -v
```

![wps19.jpg](https://ae03.alicdn.com/kf/U2a06050dbf124871a5d7e8802452d1b5N.jpg) 

 

 