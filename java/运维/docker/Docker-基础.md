## Docker简介

![1591945000290.png](https://ae02.alicdn.com/kf/Ua37e1b63c9614bc089da69372b7d3dafi.jpg)



Docker是一个开源的容器引擎，它有助于更快地交付应用。 Docker可将应用程序和基础设施层隔离，并且能将基础设施当作程 序一样进行管理。使用 Docker可更快地打包、测试以及部署应用程序，并可以缩短从编写到部署运行代码的周期。

### Docker的优点如下：

#### 1、简化程序

Docker 让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，便可以实现虚 拟化。Docker改变了虚拟化的方式，使开发者可以直接将自己的成果放入Docker中进行管理。方便快捷已经是 Docker的最大 优势，过去需要用数天乃至数周的 任务，在Docker容器的处理下，只需要数秒就能完成。

#### 2、避免选择恐惧症

如果你有选择恐惧症，还是资深患者。Docker 帮你 打包你的纠结！比如 Docker 镜像；Docker 镜像中包含了运行环境和配 置，所以 Docker 可以简化部署多种应用实例工作。比如 Web 应用、后台应用、数据库应用、大数据应用比如 Hadoop 集群、 消息队列等等都可以打包成一个镜像部署。

#### 3、节省开支

一方面，云计算时代到来，使开发者不必为了追求效果而配置高额的硬件，Docker 改变了高性能必然高价格的思维定势。 Docker 与云的结合，让云空间得到更充分的利用。不仅解决了硬件管理的问题，也改变了虚拟化的方式。

## Docker的架构

![1591945051734.png](https://ae04.alicdn.com/kf/U23cb488a080a4a0fabfb185143ee3323T.jpg)

- **Docker daemon（ Docker守护进程）**

Docker daemon是一个运行在宿主机（ DOCKER-HOST）的后台进程。可通过 Docker客户端与之通信。

- Client（ Docker客户端）

Docker客户端是 Docker的用户界面，它可以接受用户命令和配置标识，并与 Docker daemon通信。图中， docker buil

等都是 Docker的相关命令。

- **Images（ Docker镜像）**

Docker镜像是一个只读模板，它包含创建 Docker容器的说明。

它和系统安装光盘有点像

，使用系统安装光盘可以安装系

统，同理，使用Docker镜像可以运行 Docker镜像中的程序。

- **Container（容器）**

容器是镜像的可运行实例。

镜像和容器的关系有点类似于面向对象中，类和对象的关系

。可通过 Docker API或者 CLI命令来

启停、移动、删除容器。

- **Registry**

Docker Registry是一个集中存储与分发镜像的服务。构建完 Docker镜像后，就可在当前宿主机上运行。但如果想要在其他

机器上运行这个镜像，就需要手动复制。此时可借助 Docker Registry来避免镜像的手动复制。

一个 Docker Registry可包含多个 Docker仓库，每个仓库可包含多个镜像标签，每个标签对应一个 Docker镜像。这跟

Maven的仓库有点类似，如果把 Docker Registry比作 Maven仓库的话，那么 Docker仓库就可理解为某jar包的路径，而

镜像标签则可理解为jar包的版本号。

Docker Registry可分为公有Docker Registry和私有Docker Registry。 最常⽤的Docker Registry莫过于官⽅的Docker

Hub， 这也是默认的Docker Registry。 Docker Hub上存放着⼤量优秀的镜像， 我们可使⽤Docker命令下载并使⽤。

## Docker 的安装

Docker 是一个开源的商业产品，有两个版本：社区版（Community Edition，缩写为 CE）和企业版（Enterprise Edition，缩

写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到。下面的介绍都针对社区版。

Docker CE 的安装请参考官方文档，

我们这里以CentOS为例：

1、Docker 要求 CentOS 系统的内核版本高于 3.10

通过 uname -r 命令查看你当前的内核版本

```
uname ‐ r
```

2、使用 root 权限登录 Centos。确保 yum 包更新到最新。

```
yum ‐ y update
```

3、卸载旧版本(如果安装过旧版本的话)

```
yum remove docker docker‐common docker‐selinux docker‐engine
```

4、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```
yum install ‐y yum‐utils device‐mapper‐persistent‐data lvm2
```

5、设置yum源，并

更新 yum 的包索引

```
yum‐config‐manager ‐‐add‐repo http://mirrors.aliyun.com/docker‐ce/linux/centos/docker‐ce.repo


yum makecache fast
```

6、可以查看所有仓库中所有docker版本，并选择特定版本安装

```
yum list docker‐ce ‐‐showduplicates | sort‐r
```

7、安装docker

```
yum ‐y install docker‐ce‐18.03.1.ce #这是指定版本安装


yum ‐y install docker‐ce #这是安装最新稳定版
```

8、启动并加入开机启动

```
systemctl start docker


systemctl enable docker
```

9、验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

```
docker version
```

10、卸载docker

```
yum ‐y remove docker‐engine
```

## Docker常用命令

### 镜像相关命令

#### 1、搜索镜像

可使用 docker search命令搜索存放在 Docker Hub中的镜像。执行该命令后， Docker就会在Docker Hub中搜索含有 java这

个关键词的镜像仓库。

```
docker search java
```

以上列表包含五列，含义如下：

- NAME:镜像仓库名称。
- DESCRIPTION:镜像仓库描述。
- STARS：镜像仓库收藏数，表示该镜像仓库的受欢迎程度，类似于 GitHub的 stars0
- OFFICAL:表示是否为官方仓库，该列标记为[0K]的镜像均由各软件的官方项目组创建和维护。
- AUTOMATED：表示是否是自动构建的镜像仓库。

#### 2、下载镜像

使用命令docker pull命令即可从 Docker Registry上下载镜像，执行该命令后，Docker会从 Docker Hub中的 java仓

新版本的 Java镜像。如果要下载指定版本则在java后面加冒号指定版本，例如：docker pull java:8

```
docker pull java:8
```

#### 3、列出镜像

使用 docker images命令即可列出已下载的镜像

```
docker images
```

#### 4、删除本地镜像

使用 docker rmi命令即可删除指定镜像，强制删除加 -f

```
docker rmi java
```

删除所有镜像

```
docker rmi $(docker images‐q)
```

### 容器相关命令

#### 1、新建并启动容器

使用以下docker run命令即可新建并启动一个容器，该命令是最常用的命令，它有很多选项，下面将列举一些常用的选项。

-d选项：表示后台运行

-P选项：随机端口映射

-p选项：指定端口映射，有以下四种格式。

-- ip:hostPort:containerPort

-- ip::containerPort

-- hostPort:containerPort

-- containerPort

--net选项：指定网络模式，该选项有以下可选参数：

--net=bridge:

默认选项

，表示连接到默认的网桥。

--net=host:容器使用宿主机的网络。

--net=container:NAME-or-ID：告诉 Docker让新建的容器使用已有容器的网络配置。

--net=none：不配置该容器的网络，用户可自定义网络配置。

```
docker run ‐d ‐p 91:80 nginx
```

这样就能启动一个 Nginx容器。在本例中，为 docker run添加了两个参数，含义如下：

-d 后台运行

-p 宿主机端口:容器端口 #开放容器端口到宿主机端口

访问 [http://Docker宿主机](#) IP:91/，将会看到nginx的主界面如下：



#### **2、列出容器** 

用 docker ps命令即可列出**运行中**的容器 

```
docker ps 
```

如需列出所有容器（包括已停止的容器），可使用-a参数。该列表包含了7列，含义如下 

\- CONTAINER_ID：表示容器 ID。 

\- IMAGE:表示镜像名称。 

\- COMMAND：表示启动容器时运行的命令。 

\- CREATED：表示容器的创建时间。 

\- STATUS：表示容器运行的状态。UP表示运行中， Exited表示已停止。 

\- PORTS:表示容器对外的端口号。 

\- NAMES:表示容器名称。该名称默认由 Docker自动生成，也可使用 docker run命令的--name选项自行指定。 

#### **3、停止容器** 

使用 docker stop命令，即可停止容器 

```
docker stop f0b1c8ab3633 
```

其中f0b1c8ab3633是容器 ID,当然也可使用 docker stop容器名称来停止指定容器 

#### **4、强制停止容器** 

可使用 docker kill命令发送 SIGKILL信号来强制停止容器 

```
docker kill f0b1c8ab3633 
```

#### **5、****启动****已停止的容器** 

使用docker run命令，即可**新建**并启动一个容器。对于已停止的容器，可使用 docker start命令来**启动** 

```
docker start f0b1c8ab3633 

```

#### **6、查看容器所有信息** 

```
docker inspect f0b1c8ab3633 
```

#### **7、查看容器日志** 

```
docker container logs f0b1c8ab3633
```

#### **8、查看容器里的进程** 

```
docker top f0b1c8ab3633 
```

#### **9、容器与宿主机相互复制文件** 

从容器里面拷文件到宿主机： 

1 docker cp 容器id:要拷贝的文件在容器里面的路径 宿主机的相应路径 

2 如：docker cp 7aa5dc458f9d:/etc/nginx/nginx.conf /mydata/nginx 

从宿主机拷文件到容器里面： 

1 docker cp 要拷贝的宿主机文件路径 容器id:要拷贝到容器里面对应的路径 

#### **10、进入容器** 

使用docker exec命令用于进入一个正在运行的docker容器。如果docker run命令运行容器的时候，没有使用-it参数，就要用这 

个命令进入容器。一旦进入了容器，就可以在容器的 Shell 执行命令了 

```
docker exec ‐it f0b1c8ab3633 /bin/bash (有的容器需要把 /bin/bash 换成 sh) 
```

#### **11、容器内安装vim、ping、ifconfig等指令** 

1 apt‐get update 

2 apt‐get install vim #安装vim 

3 apt‐get install iputils‐ping #安装ping 

4 apt‐get install net‐tools #安装ifconfig 

#### **12、删除容器** 

使用 docker rm命令即可删除指定容器 

```
docker rm f0b1c8ab3633 
```

该命令只能删除**已停止**的容器，如需删除正在运行的容器，可使用-f参数 

强制删除所有容器 

```
docker rm ‐f $(docker ps ‐a ‐q) 
```

## **将微服务运行在docker上** 

### **使用Dockerfile构建Docker镜像** 

Dockerfile是一个文本文件，其中包含了若干条指令，指令描述了构建镜像的细节 

先来编写一个最简单的Dockerfile，以前文下载的Nginx镜像为例，来编写一个Dockerfile修改该Nginx镜像的首页 

1、新建一个空文件夹docker-demo，在里面再新建文件夹app，在app目录下新建一个名为Dockerfile的文件，在里面增加如 

下内容：

```
FROM nginx 

RUN echo '<h1>This is Tuling Nginx!!!</h1>' > /usr/share/nginx/html/index.html 
```

该Dockerfile非常简单，其中的 FROM、 RUN都是 Dockerfile的指令。 FROM指令用于指定基础镜像， RUN指令用于执行命 

令。

2、在Dockerfile所在路径执行以下命令构建镜像： 

```
docker build ‐t nginx:tuling . 
```

其中，-t指定镜像名字，命令最后的点（.）表示Dockerfile文件所在路径 

3、执行以下命令，即可使用该镜像启动一个 Docker容器 

```
docker run ‐d ‐p 92:80 nginx:tuling 
```

4、访问 http://Docker宿主机IP:92/，可看到下图所示界面

**Dockerfile常用指令** 

![1591946021381.png](https://ae02.alicdn.com/kf/U9696b922bc2646f9a83a2ebdb877f6924.jpg)

注意：RUN命令在 image 文件的构建阶段执行，执行结果都会打包进入 image 文件；CMD命令则是在容器启动后执行。另 

外，一个 Dockerfile 可以包含多个RUN命令，但是只能有一个CMD命令。 

注意，指定了CMD命令以后，docker container run命令就不能附加命令了（比如前面的/bin/bash），否则它会覆盖CMD命 

令。

### **使用Dockerfile构建微服务镜像** 

以项目**05-ms-eureka-server**为例，将该微服务的可运行jar包构建成docker镜像 

1、将jar包上传linux服务器/usr/local/docker-app/docker-demo/app/eureka目录，在jar包所在目录创建名为Dockerfile的文 

件

2、在Dockerfile中添加以下内容 

```
# 基于java镜像创建新镜像
FROM java:8
# 作者
MAINTAINER tpp
# 将jar包添加到容器中并更名为app.jar
ADD  microservice‐eureka‐serve-0.0.1-RELEASE.jar /springboot.jar
# 运行jar包
ENTRYPOINT ["nohup","java","-jar","/springboot.jar","&"]
```

3、使用docker build命令构建镜像 

```
docker build ‐t microservice‐eureka‐server:0.0.1 . 
```

\# 格式： docker build -t 镜像名称:标签 Dockerfile的相对位置 

在这里，使用-t选项指定了镜像的标签。执行该命令后，终端将会输出如下的内容4、启动镜像，加-d可在后台启动 

```
docker run ‐d ‐p 8761:8761 microservice‐eureka‐server:0.0.1 
```

使用 -v 可以挂载一个主机上的目录到容器的目录 

```
docker run ‐p 8761:8761 ‐v /log:/container‐log microservice‐eureka‐server:0.0.1 
```

5、访问http://Docker宿主机IP:8761/，可正常显示Eureka Server首页 



## **Docker虚拟化原理**

![1591946891503.png](https://ae03.alicdn.com/kf/Uab4ec5a31b60493398988e5cafb1cc38k.jpg)

传统虚拟化和容器技术结构比较：传统虚拟化技术是在硬件层面实现虚拟化，增加了系统调用链路的环节，有性能损耗；容器虚 拟化技术以共享宿主机Kernel的方式实现，几乎没有性能损耗。 docker利用的是宿主机的内核,而不需要Guest OS。因此,当新建一个容器时,docker不需要和虚拟机一样重新加载一个操作系统 内核。避免了寻址、加载操作系统内核这些比较费时费资源的过程,当新建一个虚拟机时,虚拟机软件需要加载Guest OS,这个新建 过程是分钟级别的。而docker由于直接利用宿主机的操作系统,则省略了这个过程,因此新建一个docker容器只需要几秒钟。 

![1591946936231.png](https://ae03.alicdn.com/kf/U0793f6e2bc6d4be1abbfcfdd43d0713a4.jpg)

**Docker是如何将机器的资源进行隔离的？** 

答案是联合文件系统，常见的有AUFS、Overlay、devicemapper、BTRFS和ZFS等。 

以Overlay2举例说明，Overlay2的架构图如下： 

![1591946955669.png](https://ae02.alicdn.com/kf/Uc8679a84eb154a458b6612867d62a3dd5.jpg)

原理：overlayfs在linux主机上只有两层，一个目录在下层，用来保存镜像(docker)，另外一个目录在上层，用来存储容器信 息。在overlayfs中，底层的目录叫做lowerdir，顶层的目录称之为upperdir，对外提供统一的文件系统为merged。当需要修改 一个文件时，使用**COW(Copy-on-write)**将文件从只读的Lower复制到可写的Upper进行修改，结果也保存在Upper层。在 Docker中，底下的只读层就是image，可写层就是Container。 

**写时复制 (CoW) 技术详解**所有驱动都用到的技术—写时复制，Cow全称copy-on-write，表示只是在需要写时才去复制，这个是**针对已有文件的修改场** 

**景**。比如基于一个image启动多个Container，如果每个Container都去分配一个image一样的文件系统，那么将会占用大量的磁 

盘空间。而CoW技术可以让所有的容器共享image的文件系统，所有数据都从image中读取，只有当要对文件进行写操作时，才 

从image里把要写的文件复制到自己的文件系统进行修改。所以无论有多少个容器共享一个image，所做的写操作都是对从 

image中复制到自己的文件系统的副本上进行，并不会修改image的源文件，且多个容器操作同一个文件，会在每个容器的文件 

系统里生成一个副本，每个容器修改的都是自己的副本，互相隔离，互不影响。使用CoW可以有效的提高磁盘的利用率。**所以容** 

**器占用的空间是很少的。** 

**查看容器占用磁盘大小指令：** 

```
 # 查看所有容器的大小 
 cd /var/lib/docker/containers # 进入docker容器存储目录 
 du ‐sh * # 查看所有容器的大小 
 du ‐sh <容器完整id> #查看某一个容器的大小 
```



**用时分配 （allocate-on-demand）** 

用时分配是**针对原本没有这个文件的场景**，只有在要新写入一个文件时才分配空间，这样可以提高存储资源的利用率。比如启动 一个容器，并不会因为这个容器分配一些磁盘空间，而是当有新文件写入时，才按需分配新空间。 

**docker中的镜像分层技术的原理是什么呢？** 

docker使用共享技术减少镜像存储空间，所有镜像层和容器层都保存在宿主机的文件系统/var/lib/docker/中，由存储驱动进行 管理，尽管存储方式不尽相同，但在所有版本的Docker中都可以**共享镜像层**。在下载镜像时，Docker Daemon会检查镜像中的 镜像层与宿主机文件系统中的镜像层进行对比，如果存在则不下载，只下载不存在的镜像层，这样可以非常**节约存储空间**。

![1591947065298.png](https://ae02.alicdn.com/kf/U4f54d63fbc274f1bb0aaf17964f016a8D.jpg)