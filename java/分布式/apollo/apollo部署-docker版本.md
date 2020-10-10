```
docker run -p 8080:8080 \
    -e SPRING_DATASOURCE_URL="jdbc:mysql://192.168.1.106:3306/ApolloConfigDB?characterEncoding=utf8" \
    -e SPRING_DATASOURCE_USERNAME=root -e SPRING_DATASOURCE_PASSWORD=123456 \
    -d -v /tmp/logs:/opt/logs --name apollo-configservice apolloconfig/apollo-configservice:1.7.0
```

```
docker run -p 8090:8090 \
    -e SPRING_DATASOURCE_URL="jdbc:mysql://192.168.1.106:3306/ApolloConfigDB?characterEncoding=utf8" \
    -e SPRING_DATASOURCE_USERNAME=root -e SPRING_DATASOURCE_PASSWORD=123456 \
    -d -v /tmp/logs:/opt/logs --name apollo-adminservice apolloconfig/apollo-adminservice:1.7.0
```

```
docker run -p 8070:8070 \
    -e SPRING_DATASOURCE_URL="jdbc:mysql://192.168.1.106:3306/ApolloPortalDB?characterEncoding=utf8" \
    -e SPRING_DATASOURCE_USERNAME=root -e SPRING_DATASOURCE_PASSWORD=123456 \
    -e APOLLO_PORTAL_ENVS=dev,pro \
    -e DEV_META=http://192.168.1.106:8080 -e PRO_META=http://192.168.1.106:8080 \
    -d -v /tmp/logs:/opt/logs --name apollo-portal apolloconfig/apollo-portal:1.7.0
```