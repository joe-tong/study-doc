# linux环境下搭建mysql

下载地址：https://dev.mysql.com/downloads/mysql/5.7.html#downloads

![1595818599280](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595818599280.png)

#### 解压

```
tar -xvf mysql-5.7.26-linux-glibc2.12-x86_64 /usr/local/mysql
```

####  再移动并重命名一下

```
mv mysql-5.7.26-linux-glibc2.12-x86_64 /usr/local/mysql
```

#### 创建mysql用户组和用户并修改权限

```
groupadd mysql
useradd -r -g mysql mysql
```

#### 创建数据目录并赋予权限

```
mkdir -p  /data/mysql              #创建目录
chown mysql:mysql -R /data/mysql   #赋予权限
```

#### 配置my.cnf

```
vim /etc/my.cnf
```

 内容如下:

```
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid
#character config
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
```

### 初始化数据库

#### 进入mysql的bin目录

```
cd /usr/local/mysql/bin/
```

#### 初始化

```
./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize
```

####  查看密码

```
cat /data/mysql/mysql.err
```

### 启动mysql，并更改root 密码

#### 先将mysql.server放置到/etc/init.d/mysql中

```
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
```
启动！！！
```
service mysql start

ps -ef|grep mysql
```

到这里说明mysql已经安装成功了！！

下面修改密码

首先登录mysql，前面的那个是随机生成的。

```
./mysql -u root -p   #bin目录下
```

再执行下面三步操作，然后重新登录。

```
SET PASSWORD = PASSWORD('123456');
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
FLUSH PRIVILEGES;                          
```
 这时候你如果使用远程连接……你会发现你无法连接。

这里主要执行下面三个命令(先登录数据库)

```
use mysql                                            #访问mysql库
update user set host = '%' where user = 'root';      #使root能再任何host访问
FLUSH PRIVILEGES;                                    #刷新
```
ok！！！！MySQL5.7就装好了……坑是真的多……但是如果按这个流程走应该是能顺利装下来的。（因为我装了两遍……）

如果不希望每次都到bin目录下使用mysql命令则执行以下命令

```
ln -s  /usr/local/mysql/bin/mysql    /usr/bin
```