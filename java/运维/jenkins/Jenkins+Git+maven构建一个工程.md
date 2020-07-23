# Jenkins构建Pipeline工程

## 1.添加插件

![1594885368786](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1594885368786.png)

```
1.Maven Integration plugin
2.Publish over ssh
3.pipeline
4.git parameter
```

## 2.系统配置

![1595319618408](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595319618408.png)

```
Name: 服务名称（随便写）
Hostname：主机IP
Username: 主机登录账户
Remote Directory: 远程目录
Passphrase / Password: 主机登录密码
```

## 3.创建一个Pipeline工程

#### 3.1 general配置

![1595416261957](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595416261957.png)

分支切换

#### 3.2 流水线

3.2.1 pipeline script 

```
node {
    environment {
        JENKINS_WORKSPACE = "/var/jenkins_mount/workspace"    //jenkins存放文件的地址
        PROJECT_NAME = "${JOB_NAME}"                      // 项目名
        JAR_NAME = "bnb-system-*-beta.jar"   // 项目生成的jar的名字
        VERSION_ID = "${BUILD_ID}"
    }
  
    stage('拉取代码') {
        echo 'this is pull code step'
        git branch: "${branchs}", url: 'https://github.com/joe-tong/springboot_demo2.git'
    }

    stage('编译打包') {
        echo 'this is build step'
        sh   '/var/jenkins_home/apache-maven-3.6.3/bin/mvn clean package -Dmaven.test.skip=true'
    }
    stage('推送') {
        echo 'transfer file to target server'

        sshPublisher(publishers: [
                sshPublisherDesc(configName: '本地', transfers: [
                        sshTransfer(cleanRemote: false, excludes: '',
                                execCommand:'''
                                    #!/bin/bash
                                    cd /usr/local/jenkins/workspace/target
                                    jarname=`ls -t | grep springboot*.jar | grep -v 'sources'| awk '{print $1}'|head -n 1`

                                    mv /usr/local/jenkins/workspace/target/${jarname} /usr/local/jenkins/workspace/${jarname}

                                    kill_id=`netstat -tunlp|grep 8443|awk \'{print $7}\'|awk -F \'/\' \'{print $1}\'`

                                    if [ -n "$kill_id" ]
                                    then
                                        echo "kill -9 的pid:" $kill_id
                                        kill -9 $kill_id
                                    fi
                                    sleep 1s
                                    kill_debug_id=`netstat -tunlp | grep 8060 | awk \'{print $7}\'|awk -F \'/\' \'{print $1}\'`
                                    if [ -n "$kill_debug_id" ]
                                    then
                                        echo "kill -9 kill_debug_id:" $kill_debug_id
                                        kill -9 $kill_debug_id
                                    fi
                                    sleep 1s
                                    echo "==============应用启动 --> ${jarname} =============="
                                    source /etc/profile
                                    nohup java -jar  /usr/local/jenkins/workspace/${jarname}  --spring.profiles.active=dev  >> /usr/local/jenkins/workspace/springboot.log &

                                    sleep 1s
                                    tail -f /usr/local/jenkins/workspace/springboot.log
                                '''
                                ,
                                execTimeout: 120000,
                                flatten: false,
                                makeEmptyDirs: false,
                                noDefaultExcludes: false,
                                patternSeparator: '[, ]+',
                                remoteDirectory: '',
                                remoteDirectorySDF: false,
                                removePrefix: '',
                                sourceFiles: '**/springboot_demo*.jar')],
                        usePromotionTimestamp: false,
                        useWorkspaceInPromotion: false,
                        verbose: false)
        ])

        println "==========结束==========="
    }
}

```





3.2.2 piple script from  scm





![1595320152361](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595320152361.png)

配置  git仓库

jenkinsfile 在项目根目录下

```
node {
    environment {
        JENKINS_WORKSPACE = "/var/jenkins_mount/workspace"    //jenkins存放文件的地址
        PROJECT_NAME = "${JOB_NAME}"                      // 项目名
        JAR_NAME = "bnb-system-*-beta.jar"   // 项目生成的jar的名字
        VERSION_ID = "${BUILD_ID}"
    }
    stage('拉取代码') {
        echo 'this is pull code step'
        checkout scm
    }

    stage('编译打包') {
        echo 'this is build step'
        sh   '/var/jenkins_home/apache-maven-3.6.3/bin/mvn clean package -Dmaven.test.skip=true'
    }
    stage('推送') {
        echo 'transfer file to target server'

        sshPublisher(publishers: [
                sshPublisherDesc(configName: '本地', transfers: [
                        sshTransfer(cleanRemote: false, excludes: '',
                                execCommand:'''
                                    #!/bin/bash
                                    cd /usr/local/jenkins/workspace/target
                                    jarname=`ls -t | grep springboot*.jar | grep -v 'sources'| awk '{print $1}'|head -n 1`

                                    mv /usr/local/jenkins/workspace/target/${jarname} /usr/local/jenkins/workspace/${jarname}

                                    kill_id=`netstat -tunlp|grep 8443|awk \'{print $7}\'|awk -F \'/\' \'{print $1}\'`

                                    if [ -n "$kill_id" ]
                                    then
                                        echo "kill -9 的pid:" $kill_id
                                        kill -9 $kill_id
                                    fi
                                    sleep 1s
                                    kill_debug_id=`netstat -tunlp | grep 8060 | awk \'{print $7}\'|awk -F \'/\' \'{print $1}\'`
                                    if [ -n "$kill_debug_id" ]
                                    then
                                        echo "kill -9 kill_debug_id:" $kill_debug_id
                                        kill -9 $kill_debug_id
                                    fi
                                    sleep 1s
                                    echo "==============应用启动 --> ${jarname} =============="
                                    source /etc/profile
                                    nohup java -jar  /usr/local/jenkins/workspace/${jarname}  --spring.profiles.active=dev  >> /usr/local/jenkins/workspace/springboot.log &

                                    sleep 1s
                                    tail -f /usr/local/jenkins/workspace/springboot.log
                                '''
                                ,
                                execTimeout: 120000,
                                flatten: false,
                                makeEmptyDirs: false,
                                noDefaultExcludes: false,
                                patternSeparator: '[, ]+',
                                remoteDirectory: '',
                                remoteDirectorySDF: false,
                                removePrefix: '',
                                sourceFiles: '**/springboot_demo*.jar')],
                        usePromotionTimestamp: false,
                        useWorkspaceInPromotion: false,
                        verbose: false)
        ])

        println "==========结束==========="
    }
}


```

3.3.3

```
#!/bin/sh
#docker 镜像/容器名字或者jar名字 这里都命名为这个
SERVER_NAME=springboot
#容器id
CID=$(docker ps | grep "$SERVER_NAME" | awk '{print $1}')
#镜像id
IID=$(docker images | grep "$SERVER_NAME" | awk '{print $3}')
#当前日期
DATE=`date +%Y%m%d`

#清除旧容器
if [ -n "$CID" ]; then
echo "存在$SERVER_NAME容器，CID=$CID"
echo "停止旧容器"
docker stop $SERVER_NAME
echo "删除旧容器"
docker rm $SERVER_NAME
fi

# 清楚旧镜像
if [ -n "$IID" ]; then
echo "存在$SERVER_NAME镜像，IID=$IID"
echo "删除镜像"
docker rmi $IID
fi

#构建镜像
echo "开始构建镜像"
docker build -f ./src/main/docker/Dockerfile -t $SERVER_NAME:v${DATE} ./target
echo "构建镜像成功!"


# 运行docker容器
echo "创建并启动$SERVER_NAME容器..."
docker run --name $SERVER_NAME -d -p 8080:8443 $SERVER_NAME:v${DATE}
echo "$SERVER_NAME容器启动完成"
```

