# chapter5

docker run --name nginx -d -p 8082:8082 -v /Users/didi/docker/nginx:/usr/share/nginx/html hub.c.163.com/library/nginx

docker exec -it nginx bash

apt-get update

apt-get install vim

docker commit -m "vim and  nginx" -a "vim" 1016c92378d8 vim/nginx:v1



