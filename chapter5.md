# chapter5

docker run --name nginx -d -p 8082:8082 -v /Users/didi/docker/nginx:/usr/share/nginx/html hub.c.163.com/library/nginx

docker exec -it nginx bash

apt-get update

apt-get install vim

vi /etc/nginx/conf.d/default.conf

添加php-fpm的配置：

location ~ \.php$ {

    root           /usr/share/nginx/html;

    fastcgi\_pass   192.168.59.103:9000;

    fastcgi\_index  index.php;

    fastcgi\_param  SCRIPT\_FILENAME  /var/www/html/$fastcgi\_script\_name;

    include        fastcgi\_params;

}

docker commit -m "vim and  nginx" -a "vim" 1016c92378d8 vim/nginx:v1

docker run --name php-fpm -d -p 8083:8083 -v /Users/didi/docker/nginx:/var/www/html hub.c.163.com/yswtrue/php-fpm

docker exec -it php-fpm bash

