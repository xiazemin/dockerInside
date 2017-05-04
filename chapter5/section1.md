# Nginx+PhpFpm

$ docker run --name php-fpm -d -p 8083:8083 -v /Users/didi/docker/nginx:/var/www/html hub.c.163.com/yswtrue/php-fpm

nginx 配置目录/etc/nginx/nginx.conf

$ docker run --name nginx\_fpm -d -p 8084:8084 --link php-fpm:php-fpm -v  /Users/didi/docker/nginx/:/etc/nginx/ --volumes-from php-fpm  curl/vim/nginx:v1

$ docker exec -it nginx\_fpm bash

Error response from daemon:

$ docker run --name nginx\_fpm -it -p 8084:8084 --link php-fpm:php-fpm -v  /Users/didi/docker/nginx/:/etc/nginx/ --volumes-from php-fpm  curl/vim/nginx:v1

2017/05/04 04:12:02 \[crit\] 1\#1: pread\(\) "/etc/nginx/nginx.conf" failed \(21: Is a directory\)

nginx: \[crit\] pread\(\) "/etc/nginx/nginx.conf" failed \(21: Is a directory\)

$ docker exec -it nginx bash

root@4097947fb2e8:/\# cp -r  /etc/nginx /usr/share/nginx/html/

$ docker run --name nginx\_fpm -it -p 8084:8084 --link php-fpm:php-fpm -v  /Users/didi/docker/nginx/nginx:/etc/nginx/ --volumes-from php-fpm  curl/vim/nginx:v1

$ docker exec -it nginx\_fpm bash

root@32ebf5ffc676:/\# vim /etc/nginx//conf.d/default.conf

root@32ebf5ffc676:/\# nginx -s reload



