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

