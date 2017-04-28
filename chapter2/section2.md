# docker国内镜像拉取和镜像加速registry-mirrors配置修改

由于国内访问直接访问Docker hub网速比较慢，拉取镜像的时间就会比较长。一般我们会使用镜像加速或者直接从国内的一些平台镜像仓库上拉取。

我比较常用的是网易的镜像中心和daocloud镜像市场。

网易镜像中心：[https://c.163.com/hub\#/m/home/](https://c.163.com/hub#/m/home/)

daocloud镜像市场：[https://hub.daocloud.io/](https://hub.daocloud.io/)

阿里：[https://dev.aliyun.com/search.html](https://dev.aliyun.com/search.html)

![](/assets/import1.png)$docker pull hub.c.163.com/library/mysql:latest

latest: Pulling from library/mysql

6d827a3ef358: Pull complete

ed0929eb7dfe: Pull complete

03f348dc3b9d: Pull complete

fd337761ca76: Pull complete

7e6cc16d464a: Pull complete

ca3d380bc018: Pull complete

23c12ddae61f: Pull complete

14553c628372: Pull complete

c9445076b453: Pull complete

8feabd297745: Pull complete

835cd3cde5d5: Pull complete

Digest: sha256:f9506a2e13ac2b4c0e5e1b95bbe96b154f3ae571d9eb503bec66cb105203e8ea

Status: Downloaded newer image for hub.c.163.com/library/mysql:latest

查看镜像

$  docker image ls

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE

docker-whale                  latest              ca5555513a01        2 hours ago         277 MB

xiazemin/docker-whale         latest              ca5555513a01        2 hours ago         277 MB

&lt;none&gt;                        &lt;none&gt;              26371304c9de        3 hours ago         188 MB

ubuntu                        14.04               302fa07d8117        2 weeks ago         188 MB

hub.c.163.com/library/mysql   latest              d5127813070b        2 weeks ago         407 MB

docker/whalesay               latest              6b362a9f73eb        23 months ago       247 MB

启动容器：运行镜像

$ docker run --name some-mysql -e MYSQL\_ROOT\_PASSWORD=my-secret-pw -d hub.c.163.com/library/mysql

edf73a1d208d5155dcdb5f97f452b376f3181298ad2ec547a4ae60512401d860

$    docker ps

CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS              PORTS               NAMES

edf73a1d208d        hub.c.163.com/library/mysql   "docker-entrypoint..."   6 minutes ago       Up 6 minutes        3306/tcp            some-mysql

$ docker start edf73a1d208d

edf73a1d208d

连接Mysql数据库

\(1\).客户端工具连接

我这里用mysql的可视化工具workbench连接db。

workbench下载地址：[http://dev.mysql.com/downloads/workbench/](http://dev.mysql.com/downloads/workbench/)

\(2\).docker下命令行连接

1\).首先，进入CMD执行下列命令

docker exec -it edf73a1d208d bash

root@edf73a1d208d:/\#

2\).然后，输入下面命令，并输入密码password

my-secret-pw

$  docker exec -it edf73a1d208d bash

root@edf73a1d208d:/\# mysql -uroot -p -h localhost

Enter password:

Welcome to the MySQL monitor.  Commands end with ; or \g.

Your MySQL connection id is 3

Server version: 5.7.18 MySQL Community Server \(GPL\)



Copyright \(c\) 2000, 2017, Oracle and/or its affiliates. All rights reserved.



Oracle is a registered trademark of Oracle Corporation and/or its

affiliates. Other names may be trademarks of their respective

owners.



Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.



mysql&gt;

