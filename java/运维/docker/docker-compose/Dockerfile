FROM java:8
MAINTAINER 同平平
ADD  zoo-0.0.1-SNAPSHOT.jar /root/docker/zoo.jar
expose 9099
ENTRYPOINT ["nohup","java","-jar","/root/docker/zoo.jar","&"]