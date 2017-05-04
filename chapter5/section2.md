# LNMP

$    docker pull hub.c.163.com/mrjucn/centos6.5-mysql5.1-php5.7-nginx:latest

latest: Pulling from mrjucn/centos6.5-mysql5.1-php5.7-nginx

9110a7be4103: Pull complete

a3ed95caeb02: Pull complete

8a1dcc3f76c2: Pull complete

ce4980537143: Pull complete

43fc3558431f: Pull complete

5881bc109689: Pull complete

dd6d2de68400: Pull complete

Digest: sha256:4548e5ff8b50da8d6b442cfcd0a71ec90809d1fdfe82949ffd6362b415d7c6e9

Status: Downloaded newer image for hub.c.163.com/mrjucn/centos6.5-mysql5.1-php5.7-nginx:latest

$ docker run --name lnmp -it -p 8085:8085 hub.c.163.com/mrjucn/centos6.5-mysql5.1-php5.7-nginx:latest /bin/bash

\[root@03949f263d85 /\]\# lnmp start

+-------------------------------------------+

\|    Manager for LNMP, Written by Licess    \|

+-------------------------------------------+

\|              [http://lnmp.org](http://lnmp.org)              \|

+-------------------------------------------+

Starting LNMP...

Starting nginx...  done

Starting MySQL.                                            \[  OK  \]

Starting php-fpm  done

\[root@03949f263d85 /\]\#

$nginx didi$ docker run --name lnmp -it -p 8085:80 hub.c.163.com/mrjucn/centos6.5-mysql5.1-php5.7-nginx:latest /bin/bash

![](/assets/importlnmp.png)![](/assets/importp.png)修改MySQL的登录设置：

\# vi /etc/my.cnf

在\[mysqld\]的段中加上一句：skip-grant-tables

例如：

\[mysqld\]

datadir=/var/lib/mysql

socket=/var/lib/mysql/mysql.sock

skip-grant-tables

\[root@cc11423aa34a phpmyadmin\]\# vi /etc/my.cnf

\[root@cc11423aa34a phpmyadmin\]\#

\[root@cc11423aa34a phpmyadmin\]\# /etc/init

init/   init.d/

\[client\]

\[root@cc11423aa34a phpmyadmin\]\# /etc/init.d/

atd          killall      network      portreserve  saslauthd

crond        messagebus   nginx        rdisc        sendmail

cups         mysql        ntpd         restorecond  single

halt         netconsole   ntpdate      rsyslog      sshd

iptables     netfs        php-fpm      sandbox      udev-post

\[root@cc11423aa34a phpmyadmin\]\# /etc/init.d/mysql restart

Shutting down MySQL...                                     \[  OK  \]

Starting MySQL.                                            \[  OK  \]

\[root@cc11423aa34a phpmyadmin\]\# /usr/bin/my

myisamchk    mysql        mysqlcheck   mysqld\_safe  mysqldump

\[root@cc11423aa34a phpmyadmin\]\# /usr/bin/mysql

Welcome to the MySQL monitor.  Commands end with ; or \g.

Your MySQL connection id is 1

Server version: 5.1.73-log Source distribution

Copyright \(c\) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its

affiliates. Other names may be trademarks of their respective

owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql&gt; use mysql

Database changed

mysql&gt;  UPDATE user SET Password= password\('123456'\) WHERE User = 'root' ;

Query OK, 2 rows affected \(0.00 sec\)

Rows matched: 2  Changed: 2  Warnings: 0

mysql&gt; quit

Bye

\[root@cc11423aa34a phpmyadmin\]\# vi /etc/my.cnf

\[root@cc11423aa34a phpmyadmin\]\# /etc/init.d/mysql restart

Shutting down MySQL...                                     \[  OK  \]

Starting MySQL.                                            \[  OK  \]

\[root@cc11423aa34a var\]\# /usr/bin/mysql -c -A -h127.0.0.1 -P3306 -uroot -p123456

Welcome to the MySQL monitor.  Commands end with ; or \g.

Your MySQL connection id is 1

mysql&gt;

