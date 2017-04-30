# php-fpm

$ docker pull hub.c.163.com/yswtrue/php-fpm:latest

latest: Pulling from yswtrue/php-fpm

75a822cd7888: Pulling fs layer

e4d8a4e038be: Pulling fs layer

81d4d961577a: Pulling fs layer

56da5951c6dd: Pulling fs layer

a8e2e133211c: Pulling fs layer

80bd3027ec02: Pulling fs layer

643ac79e86d7: Pull complete

58f6206070dd: Pull complete

fc237f3a560a: Pull complete

0a17cb1bb61f: Pull complete

06ce481db2f7: Pull complete

a6abd4517b2e: Pull complete

dbcead3197fd: Pull complete

bfe1132fa498: Pull complete

83db0e8e1dae: Pull complete

2ba0e7b58c8d: Pull complete

Digest: sha256:a81454cf20e8af62317a7aaf1e312c125bb7a966a1444fe97cdcabe44e3ff315

Status: Downloaded newer image for hub.c.163.com/yswtrue/php-fpm:latest

运行php

$ docker run -p 9000:9000 --name  myphp-fpm -v ~/nginx/www:/www -v $PWD/c hub.c.163.com/library/php -v $PWD/logs:/phplogs   -d hub.c.163.com/yswtrue/php-fpm

PHP 7.1.4 \(cli\) \(built: Apr 18 2017 19:12:23\) \( NTS \)

Copyright \(c\) 1997-2017 The PHP Group

Zend Engine v3.1.0, Copyright \(c\) 1998-2017 Zend Technologies



