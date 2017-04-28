# section6

 $ docker login --username=xiazemin --email=465474307@qq.com

Flag --email has been deprecated, will be removed in 1.14.

Password:

Login Succeeded

 $

FROM docker/whalesay:latest

 $

 $ ls

Dockerfile

 $ mkdir mysql

 $ cd mysql/

localhost:mysql didi$ vi Dockerfile

localhost:mysql didi$

localhost:mysql didi$ docker build -t "xiazemin/mysql-osx:latest" .

Sending build context to Docker daemon 2.048 kB

Step 1/8 : FROM mysql:latest

Get [https://registry-1.docker.io/v2/](https://registry-1.docker.io/v2/): net/http: TLS handshake timeout

localhost:mysql didi$     cd ..

 $ ls

Dockerfile    mysql

 $ vi Dockerfile

 $

 $



 $

 $

 $

 $ docker tag 8e15421920b1 xiazemin/docker-whale:latest

Error response from daemon: no such id: 8e15421920b1

 $ docker tag ca5555513a01 xiazemin/docker-whale:latest

 $ docker push xiazemin/docker-whale

The push refers to a repository \[docker.io/xiazemin/docker-whale\]

dd1361ddca89: Pushing 7.301 MB/29.64 MB

5f70bf18a086: Mounted from docker/whalesay

d061ee1340ec: Mounted from docker/whalesay

d511ed9e12e1: Mounted from docker/whalesay

091abc5148e4: Mounted from docker/whalesay

b26122d57afa: Mounted from docker/whalesay

37ee47034d9b: Mounted from docker/whalesay

528c8710fd95: Mounted from docker/whalesay

1154ba695078: Mounted from docker/whalesay

