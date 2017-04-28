# Docker创建本地mysql镜像

下载文件

$ git clone [https://github.com/xiazemin/mysql-1.git](https://github.com/xiazemin/mysql-1.git)

Cloning into 'mysql'...

remote: Counting objects: 19, done.

remote: Compressing objects: 100% \(15/15\), done.

remote: Total 19 \(delta 4\), reused 19 \(delta 4\), pack-reused 0

Unpacking objects: 100% \(19/19\), done.

Checking connectivity... done.

创建镜像

 docker build -t mysql .

Sending build context to Docker daemon 95.23 kB

Sending build context to Docker daemon

Step 0 : FROM sshd

 ---&gt; 312c93647dc3

Step 1 : MAINTAINER Waitfish &lt;dwj\_zz@163.com&gt;

 ---&gt; Running in a149f8a7933f

 ---&gt; edbbfe8b4895

Removing intermediate container a149f8a7933f

Step 2 : ENV DEBIAN\_FRONTEND noninteractive

 ---&gt; Running in e80cbb29cadb

 ---&gt; 81fc6101a236

Removing intermediate container e80cbb29cadb

Step 3 : RUN apt-get update &&   apt-get -yq install mysql-server-5.6 pwgen &&   rm -rf /var/lib/apt/lists/\*

 ---&gt; Running in 5d220fe833c2

...

Removing intermediate container 3c3254e8cc1e

Successfully built f008f97bdc14

dwj@iZ23pznlje4Z:~/mysql$ sudo docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE

mysql               latest              f008f97bdc14        About a minute ago   539.1 MB

使用示范：

不添加环境变量，使用默认方式启动容器，并映射 22 3306端口。

$ sudo docker run -d -P mysql

检查容器进程启动情况。

$ sudo docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                            NAMES

eef1632ccd4e        mysql:latest        "/run.sh"           8 seconds ago       Up 8 seconds        0.0.0.0:49153-&gt;22/tcp, 0.0.0.0:49154-&gt;3306/tcp   angry\_einstein



$ ssh 127.0.0.1 -p 49153

The authenticity of host '\[127.0.0.1\]:49153 \(\[127.0.0.1\]:49153\)' can't be established.

ECDSA key fingerprint is db:35:7a:60:2d:11:d5:97:5a:e6:84:a6:95:f0:4f:32.

Are you sure you want to continue connecting \(yes/no\)? yes

Warning: Permanently added '\[127.0.0.1\]:49153' \(ECDSA\) to the list of known hosts.

Welcome to Ubuntu 14.04 LTS \(GNU/Linux 3.2.0-54-generic x86\_64\)



 \* Documentation:  https://help.ubuntu.com/



The programs included with the Ubuntu system are free software;

the exact distribution terms for each program are described in the

individual files in /usr/share/doc/\*/copyright.



Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by

applicable law.



root@eef1632ccd4e:~\# ps -ef \|grep mysql

root         1     0  0 20:14 ?        00:00:00 /bin/sh /usr/bin/mysqld\_safe

mysql     1974     1  0 20:14 ?        00:00:00 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/mysql/plugin --user=mysql --log-error=/var/log/mysql/error.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/run/mysqld/mysqld.sock --port=3306

root      2022  2010  0 20:15 pts/0    00:00:00 grep --color=auto mysql

MySQL 的 root 用户默认没有密码只能本地访问。

mysql&gt; select host, user, password from mysql.user;

+--------------+-------+-------------------------------------------+

\| host         \| user  \| password                                  \|

+--------------+-------+-------------------------------------------+

\| localhost    \| root  \|                                           \|

\| eef1632ccd4e \| root  \|                                           \|

\| 127.0.0.1    \| root  \|                                           \|

\| ::1          \| root  \|                                           \|

\| localhost    \|       \|                                           \|

\| eef1632ccd4e \|       \|                                           \|

\| %            \| admin \| \*ADDD6793DD97A040C9B039F72682E5AA31A92C35 \|

+--------------+-------+-------------------------------------------+

7 rows in set \(0.00 sec\)

拥有远程访问权限的 admin 用户的密码，可以使用 Docker logs + id 来获取。

$ sudo docker logs eef

=&gt; An empty or uninitialized MySQL volume is detected in /var/lib/mysql

=&gt; Installing MySQL ...

=&gt; Done!

=&gt; Creating admin user ...

=&gt; Waiting for confirmation of MySQL service startup, trying 0/13 ...

=&gt; Creating MySQL user admin with random password

=&gt; Done!

========================================================================

You can now connect to this MySQL Server using:



    mysql -uadmin -pt1FWuDCgQicT -h&lt;host&gt; -P&lt;port&gt;



Please remember to change the above password as soon as possible!

MySQL user 'root' has no password but only allows local connections

========================================================================

141106 20:14:21 mysqld\_safe Can't log to error log and syslog at the same time.  Remove all --log-error configuration options for --syslog to take effect.

141106 20:14:21 mysqld\_safe Logging to '/var/log/mysql/error.log'.

141106 20:14:21 mysqld\_safe Starting mysqld daemon with databases from /var/lib/mysql

上面的 t1FWuDCgQicT 就是 admin 的密码。

给 admin 用户指定用户名和密码

$ sudo docker run -d -P -e MYSQL\_PASS="mypass" mysql

1b32444ebb7232f885961faa15fb1a052ca93b81c308cc41d16bd3d276c77d75

将宿主主机的文件夹挂载到容器的数据库文件夹

默认情况数据库的数据库文件和日志文件都会存在容器的 AUFS 层，这不仅使得容器变得越来越臃肿，不便于迁移、备份等管理，而且数据库的 IOPS 也会受到影响。

$ docker run -d -P -v /opt/mysqldb:/var/lib/mysql mysql

这样，容器就会将数据文件和日志文件都放到你指定的 主机目录下面。

$ tree /opt/mysqldb/

/opt/mysqldb/

\|-- auto.cnf

\|-- ib\_logfile0

\|-- ib\_logfile1

\|-- ibdata1

\|-- mysql

\|   \|-- columns\_priv.MYD

\|   \|-- columns\_priv.MYI

\|   \|-- columns\_priv.frm

\|   \|-- db.MYD

\|   \|-- db.MYI

\|   \|-- db.frm

\|   \|-- event.MYD

\|   \|-- event.MYI

\|   \|-- event.frm

\|   \|-- func.MYD

\|   \|-- func.MYI

\|   \|-- func.frm

\|   \|-- general\_log.CSM

...

使用主从复制模式

创建一个叫 mysql 的容器。

$ docker run -d -e REPLICATION\_MASTER=true  -P  --name mysql  mysql

创建从 mysql 容器，并连接到刚刚创建的主容器。

$ docker run -d -e REPLICATION\_SLAVE=true -P  --link mysql:mysql  mysql

注意：这里的主 mysql 服务器的名字必须叫 mysql，否则会提示 \`Cannot configure slave, please link it to another MySQL Container with alias as 'mysql'！

\# docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS                                            NAMES

a781d1c74024        mysql:latest        "/run.sh"           About a minute ago   Up About a minute   0.0.0.0:49167-&gt;22/tcp, 0.0.0.0:49168-&gt;3306/tcp   romantic\_fermi

38c73b5555aa        mysql:latest        "/run.sh"           About a minute ago   Up About a minute   0.0.0.0:49165-&gt;22/tcp, 0.0.0.0:49166-&gt;3306/tcp   mysql

现在，你就可以通过相应的端口来连接主或者从 mysql 服务器了。

