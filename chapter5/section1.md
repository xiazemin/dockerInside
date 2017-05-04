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

![](/assets/importng.png)172.17.0.1 - - \[04/May/2017:05:27:39 +0000\] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_11\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/58.0.3029.96 Safari/537.36" "-"

![](/assets/importfpm.png)2017/05/04 05:28:00 \[error\] 14\#14: \*7 connect\(\) failed \(111: Connection refused\) while connecting to upstream, client: 172.17.0.1, server: localhost, request: "GET /phpinfo.php HTTP/1.1", upstream: "fastcgi://172.17.0.9:8083", host: "127.0.0.1:8084"

172.17.0.1 - - \[04/May/2017:05:28:00 +0000\] "GET /phpinfo.php HTTP/1.1" 502 537 "-" "Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_11\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/58.0.3029.96 Safari/537.36" "-"

![](/assets/importfpm1.png)$ docker exec -it php-fpm bash

root@63a3afd2b7ec:/var/www/html\# php-fpm -i



