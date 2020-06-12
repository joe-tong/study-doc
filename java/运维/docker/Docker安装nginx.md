### Docker安装nginx

1.docker pull nginx

2.docker run -d -p 80:80 --name nginx nginx:latest

3.mkdir -p /usr/local/nginx/www /usr/local/nginx/logs /usr/local/nginx/conf

4. docker cp nginx:/etc/nginx/nginx.conf /usr/local/nginx/conf
5. docker stop nginx
6. docker rm nginx
7. docker run -d -p 80:80 --name nginx -v /usr/local/nginx/www:/usr/share/nginx/html -v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /usr/local/nginx/logs:/var/log/nginx --restart=always nginx:latest

