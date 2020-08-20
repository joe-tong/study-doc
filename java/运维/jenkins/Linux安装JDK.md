# Linux安装JDK

## 一、JDK介绍

 ![](https://ftp.bmp.ovh/imgs/2020/08/62e92f721fb92cb2.jpg) 

 

　　JDK是 Java 语言的软件开发工具包，主要用于移动设备、嵌入式设备上的java应用程序。JDK是整个java开发的核心，它包含了JAVA的运行环境（JVM+Java系统类库）和JAVA工具。

## 二、准备工作（以下步骤自己有可以跳过）

　　1、jdk1.8安装包，翻到最下面去下载，系统采用CentOS 7。

## 三、开始安装

1、检查下系统中的jdk版本：

```
java -version
```

.2、检查jdk自带安装包：

```
rpm -qa | grep java
```

3、如果有自带jdk，则卸载旧的：

```
rpm -e --nodeps 要卸载的包(包通过rpm -qa | grep java获取))

或者通过以下命令卸载：

yum remove *openjdk*
```

4.1、在线下载JDK安装包，下载到 /usr/local/目录下存jdk压缩包

```
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm 

```

 ![](https://ftp.bmp.ovh/imgs/2020/08/e5b92f4a25249f29.jpg)  

 

 

 　　注意：如果在线无法下载可以直接翻到最下面关注公众号回复 ”jdk下载“ 

5、rpm安装下载好安装包

```
rpm -i jdk-8u131-linux-x64.rpm
```

 ![](https://ftp.bmp.ovh/imgs/2020/08/b2cca30f67c5ca4a.jpg)  

6、修改环境变量

```
vi /etc/profile
```

6.1、在文件profile的最后一行加上，注意：JAVA_HOME=对应的是自己的jdk存放路径，编辑后保存并退出

```
JAVA_HOME=/usr/java/jdk1.8.0_131
CLASSPATH=%JAVA_HOME%/lib:%JAVA_HOME%/jre/lib
PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
export PATH CLASSPATH JAVA_HOME
```

7、刷新配置文件

```
source /etc/profile
```

8、命令行输入java -version，出现以下界面，说明安装成功：

```
java -version
```

 ![](https://ftp.bmp.ovh/imgs/2020/08/ea3b44bad2c054e4.jpg)  

 

