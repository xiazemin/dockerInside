# Nginx+PhpFpm

$ docker run --name php-fpm -d -p 8083:8083 -v /Users/didi/docker/nginx:/var/www/html hub.c.163.com/yswtrue/php-fpm

nginx 配置目录/etc/nginx/nginx.conf

$ docker run --name nginx\_fpm -d -p 8084:8084 --link php-fpm:php-fpm -v  /Users/didi/docker/nginx/:/etc/nginx/ --volumes-from php-fpm  curl/vim/nginx:v1

