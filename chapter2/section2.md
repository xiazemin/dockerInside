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

