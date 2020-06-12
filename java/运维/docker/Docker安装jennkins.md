## Docker安装jennkins

1.docker pull jenkins

2.mkdir /home/var/jenkins -p

3.chown -R 1000:1000 jenkins/

4.docker run -itd -p 8081:8080 -p 50000:50000 --name jenkins --privileged=true  -v /home/var/jenkins:/var/jenkins_home jenkins

5.docker restart jenkins

6.docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword