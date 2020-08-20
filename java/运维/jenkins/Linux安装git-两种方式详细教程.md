# Linux安装git-两种方式详细教程

## 一、Git介绍

Git --- The stupid content tracker, 傻瓜内容跟踪器。Linus Torvalds 是这样给我们介绍 Git 的。

Git 是用于 Linux[内核](https://baike.baidu.com/item/%E5%86%85%E6%A0%B8)开发的[版本控制](https://baike.baidu.com/item/%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6)工具。与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持（wingeddevil注：这得分是用什么样的服务端，使用http协议或者git协议等不太一样。并且在push和pull的时候和服务器端还是有交互的。），使[源代码](https://baike.baidu.com/item/%E6%BA%90%E4%BB%A3%E7%A0%81)的发布和交流极其方便。 Git 的速度很快，这对于诸如 Linux kernel 这样的大项目来说自然很重要。 Git 最为出色的是它的合并跟踪（merge tracing）能力。

## 二、准备工作

.Git是目前流行的非常好用的版本控制工具，这里介绍两种安装方式，1、yum安装，2、从github上下载最新的源码编译后安装(注意：如果不会下载翻最下面)

## 三、yun安装

1、在Linux上是有yum安装Git，非常简单，只需要一行命令

```
yum -y install git
```

 ![](https://ftp.bmp.ovh/imgs/2020/08/17b58a4dc2b8229a.jpg) 

 2.输入 git --version查看Git是否安装完成以及查看其版本号

```
 git --version
```

 ![](https://ftp.bmp.ovh/imgs/2020/08/69b7a75474298ebc.jpg)  



## 四、从GitHub上下载最新的源码编译后安装

有人想问，直接在线安装多么容易，为啥还下载安装呢，你们也看到了，上述的安装版本不是Git官方最新的包，下载包安装可以选版本。

1.首先我们需要删除旧的Git

```
yum -y remove git
```

2.进入git在GitHub上发布版本页面https://github.com/git/git/releases，这个页面我们可以找到所有git已发布的版本。这里我们选择最新版的tar.gz包。

https://github.com/git/git/releases

![wps3.jpg](https://ae02.alicdn.com/kf/Ud9d2ed6ed2134242948759313f134028p.jpg) 

 

3.下载最新版本的tar.gz的Git到本地电脑上，利用Xftp工具将压缩包上传至Linux服务器的/usr/local目录下

![wps4.jpg](https://ae04.alicdn.com/kf/U546ecdc12077462a8051293fc195c64aU.jpg) 

 

4.进入/usr/local 目录解压git文件

```
tar -zxvf git-2.25.4.tar.gz
```

![wps2.jpg](https://ae02.alicdn.com/kf/Ue31a0034378441e7bacfaf0f288924deJ.jpg) 

5.拿到解压后的源码以后我们需要编译源码了，不过在此之前需要安装编译所需要的依赖。

```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```

 ![](https://ftp.bmp.ovh/imgs/2020/08/41a6432e17a361cd.jpg) 

 

6.编译git源码，进入cd /usr/local/git-2.25.4 目录

```
make prefix=/usr/local/git all
```

7.安装git至/usr/bin/git路径

```
make prefix=/usr/local/git install
```

8.配置环境变量

```
vi /etc/profile 
```

9.在底部加上如下

```
export PATH=$PATH:/usr/local/git/bin
```

10.刷新环境变量

```
source /etc/profile
```

11.查看Git是否安装完成

```
git --version
```

至此，从github上下载最新的源码编译后安装git完成。

 