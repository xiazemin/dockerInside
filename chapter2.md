# dockerhub镜像Docker-创建一个mysql容器，并保存为本地镜像

**查找docker hub上的镜像**

$ docker search mysql

NAME                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED

mysql                           MySQL is a widely used, open-source relati...   4254      \[OK\]

mysql/mysql-server              Optimized MySQL Server Docker images. Crea...   290                  \[OK\]

centurylink/mysql               Image containing mysql. Optimized to be li...   51                   \[OK\]

zabbix/zabbix-server-mysql      Zabbix Server with MySQL database support       37                   \[OK\]

zabbix/zabbix-web-nginx-mysql   Zabbix frontend based on Nginx web-server ...   18                   \[OK\]

imega/mysql                     Size: 149 MB, alpine:3.5, Mysql Server: 10...   11                   \[OK\]

appcontainers/mysql             Centos/Debian Based Customizable MySQL Con...   8                    \[OK\]

marvambass/mysql                MySQL Server based on Ubuntu 14.04              7                    \[OK\]

zabbix/zabbix-proxy-mysql       Zabbix proxy with MySQL database support        7                    \[OK\]

bitnami/mysql                   Bitnami MySQL Docker Image                      5                    \[OK\]

dnhsoft/mysql-utf8              Inherits the official MySQL image configur...   5                    \[OK\]

debezium/example-mysql          Example MySQL database server with a simpl...   3                    \[OK\]

frodenas/mysql                  A Docker Image for MySQL                        3                    \[OK\]

alterway/mysql                  Docker Mysql                                    3                    \[OK\]

drupaldocker/mysql              MySQL for Drupal                                2                    \[OK\]

yfix/mysql                      Yfix docker built mysql                         2                    \[OK\]

coscale/mysql                   CoScale custom configuration of the offici...   1                    \[OK\]

tozd/mysql                      MySQL \(MariaDB fork\) Docker image.              1                    \[OK\]

lysender/mysql                  MySQL base image using Ubuntu 16.04 Xenial      1                    \[OK\]

captomd/mysql                   CaptoMD mysql configuration                     0                    \[OK\]

projectomakase/mysql            Docker image for MySQL                          0                    \[OK\]

nanobox/mysql                   MySQL service for nanobox.io                    0                    \[OK\]

datajoint/mysql                 MySQL image pre-configured to work smoothl...   0                    \[OK\]

1maa/mysql                      MySQL database                                  0                    \[OK\]

cloudposse/mysql                Improved \`mysql\` service with support for ...   0

**下载镜像到本地**

$ docker pull mysql

Using default tag: latest

latest: Pulling from library/mysql

cd0a524342ef: Downloading 4.325 MB/52.55 MB

d9c95f06c17e: Download complete

46b2d578f59a: Download complete

**创建容器**

docker run --name mysqldb -e MYSQL\_ROOT\_PASSWORD=root -d mysql

**得到mysql镜像的IP**

docker inspect mysqldb\|grep IPAddress

"IPAddress": "172.17.0.4",

```
    "SecondaryIPAddresses": null
```

**连接mysql**

mysql -h 172.17.0.4 -u root -p

**将初始化好的mysql保存为镜像**

docker commit mysqldb mysql:1.0

3ed4a367c21eb509f1c4e0a772c3e5bdff678497be55700ea256ef34ad87cfc6

