# 安装docker

1，[Boot2Docker](https://github.com/boot2docker/boot2docker)（官方已废弃）

```
    Docker引擎使用了Linux内核特定的特性，所以要让它运行在OS X上我们需要用一个轻量型的虚拟机\(vm\)。用OS X的Docker客户端来控制虚拟Docker来构建，运行以及管理Docker容器。为了使过程更简单一点，设计了一个叫做[Boot2Docker](https://github.com/boot2docker/boot2docker)的帮助应用程序，它能按照虚拟机以及运行Docker后台程序。

 $boot2docker init

 $ boot2docker start

 $ export DOCKER\_HOST=tcp://$\(boot2docker ip 2&gt;/dev/null\):2375
```

完成虚拟化环境搭建。

$ docker run ubuntu echo hello world

下载ubuntu镜像并打印hello world。

2，docker-for-mac

下载地址：[https://docs.docker.com/docker-for-mac/](https://docs.docker.com/docker-for-mac/)

如果安装失败：

$ docker ps

-bash: docker: command not found

 $ docker run -d -p 80:80

-bash: docker: command not found


如果安装成功：
 $ docker info

Containers: 0

Running: 0

Paused: 0

Stopped: 0

Images: 0

Server Version: 17.03.1-ce

Storage Driver: overlay2

Backing Filesystem: extfs

Supports d\_type: true

Native Overlay Diff: true

Logging Driver: json-file

Cgroup Driver: cgroupfs

Plugins:

Volume: local

Network: bridge host ipvlan macvlan null overlay

Swarm: inactive

Runtimes: runc

Default Runtime: runc

Init Binary: docker-init

containerd version: 4ab9917febca54791c5f071a9d1f404867857fcc

runc version: 54296cf40ad8143b62dbcaa1d90e520a2136ddfe

init version: N/A \(expected: 949e6facb77383876aeff8a6944dde66b3089574\)

Security Options:

seccomp

Profile: default

Kernel Version: 4.9.13-moby

Operating System: Alpine Linux v3.5

OSType: linux

Architecture: x86\_64

CPUs: 2

Total Memory: 1.952 GiB

Name: moby

ID: MTBS:V3B2:D4PM:4G3G:RRI6:HFDV:PJ73:A6DP:7KYA:A2G7:AHG6:K4JX

Docker Root Dir: /var/lib/docker

Debug Mode \(client\): false

Debug Mode \(server\): true

File Descriptors: 16

Goroutines: 26

System Time: 2017-04-28T08:28:21.556923057Z

EventsListeners: 1

Http Proxy: 127.0.0.1:8888

Https Proxy: 127.0.0.1:8888

Registry: [https://index.docker.io/v1/](https://index.docker.io/v1/)

Experimental: true

Insecure Registries:

127.0.0.0/8

Live Restore Enabled: false

 $ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

 $ docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

 $  docker run hello-world

Unable to find image 'hello-world:latest' locally

docker: Error response from daemon: Get [https://registry-1.docker.io/v2/](https://registry-1.docker.io/v2/): http: error connecting to proxy [http://127.0.0.1:8888](http://127.0.0.1:8888): dial tcp 127.0.0.1:8888: getsockopt: connection refused.

See 'docker run --help'.

发现失败，重试仍然失败

 $  docker run tag

Unable to find image 'tag:latest' locally

docker: Error response from daemon: Get [https://registry-1.docker.io/v2/](https://registry-1.docker.io/v2/): http: error connecting to proxy [http://127.0.0.1:8888](http://127.0.0.1:8888): dial tcp 127.0.0.1:8888: getsockopt: connection refused.

See 'docker run --help'.

 $   docker run -d -p 80:80 --name webserver nginx

Unable to find image 'nginx:latest' locally

docker: Error response from daemon: Get [https://registry-1.docker.io/v2/](https://registry-1.docker.io/v2/): http: error connecting to proxy [http://127.0.0.1:8888](http://127.0.0.1:8888): dial tcp 127.0.0.1:8888: getsockopt: connection refused.

See 'docker run --help'.

 $   docker run -d -p 80:80 --name webserver nginx

docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.

See 'docker run --help'.

 $

 $ docker ps

Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

 $    docker run docker/whalesay cowsay boo

Unable to find image 'docker/whalesay:latest' locally

docker: Error response from daemon: Get [https://registry-1.docker.io/v2/](https://registry-1.docker.io/v2/): http: error connecting to proxy [http://127.0.0.1:8888](http://127.0.0.1:8888): dial tcp 127.0.0.1:8888: getsockopt: connection refused.

See 'docker run --help'.

 $

 $    docker run docker/whalesay cowsay boo

Unable to find image 'docker/whalesay:latest' locally

latest: Pulling from docker/whalesay

e190868d63f8: Downloading 11.35 MB/65.77 MB

909cd34c6fd7: Download complete

e190868d63f8: Pull complete

909cd34c6fd7: Pull complete

0b9bfabab7c1: Pull complete

a3ed95caeb02: Pull complete

00bf65475aba: Pull complete

c57b6bcc83e3: Pull complete

8978f6879e2f: Pull complete

8eed3712d2cf: Pull complete

Digest: sha256:178598e51a26abbc958b8a2e48825c90bc22e641de3d31e18aaf55f3258ba93b

Status: Downloaded newer image for docker/whalesay:latest

\_\_\_\_\_

&lt; boo &gt;

---

```
\

 \

  \

                \#\#        .

          \#\# \#\# \#\#       ==

       \#\# \#\# \#\# \#\#      ===

   /""""""""""""""""\_\_\_/ ===
```

~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~

```
   \\_\_\_\_\_\_ o          \_\_/

    \    \        \_\_/

      \\_\_\_\_\\_\_\_\_\_\_/
```

 $

 $

 $                      docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

docker/whalesay     latest              6b362a9f73eb        23 months ago       247 MB

 $   ls

Applications        goLang

Desktop            index.html

Documents        info.php

Downloads        keras

GitBook            nginx

Library            node

Movies            octave

Music            progress.php

PHP            python

PhpstormProjects    scala

Pictures        scikit\_learn\_data

Public            shell

SVN            shiff.php

anaconda        test.php

apcu-5.1.7.tgz        testArray.php

bashrc.bak        vpworkspace

c            项目相关文档

docker

 $

FROM mysql:latest

 $

 $ cd docker/

localhost:docker didi$ docker login --username=xiazemin --email=465474307@qq.com

Flag --email has been deprecated, will be removed in 1.14.

Password:

Login Succeeded

localhost:docker didi$

FROM docker/whalesay:latest

localhost:docker didi$

localhost:docker didi$ ls

Dockerfile

localhost:docker didi$ mkdir mysql

localhost:docker didi$ cd mysql/

localhost:mysql didi$ vi Dockerfile

localhost:mysql didi$

localhost:mysql didi$ docker build -t "xiazemin/mysql-osx:latest" .

Sending build context to Docker daemon 2.048 kB

Step 1/8 : FROM mysql:latest

Get [https://registry-1.docker.io/v2/](https://registry-1.docker.io/v2/): net/http: TLS handshake timeout

localhost:mysql didi$     cd ..

localhost:docker didi$ ls

Dockerfile    mysql

localhost:docker didi$ vi Dockerfile

localhost:docker didi$

localhost:docker didi$

localhost:docker didi$ docker build -t docker-whale .

Sending build context to Docker daemon 3.584 kB

Step 1/3 : FROM docker/whalesay:latest

---&gt; 6b362a9f73eb

Step 2/3 : RUN apt-get -y update && apt-get install -y fortunes

---&gt; Running in 2a0fb37454de

Ign [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty InRelease

Get:1 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-updates InRelease \[65.9 kB\]

Get:2 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-security InRelease \[65.9 kB\]

Hit [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty Release.gpg

Get:3 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-updates/main Sources \[491 kB\]

Get:4 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-updates/restricted Sources \[6467 B\]

Get:5 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-updates/universe Sources \[226 kB\]

Get:6 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-updates/main amd64 Packages \[1226 kB\]

Get:7 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-updates/restricted amd64 Packages \[21.2 kB\]

Get:8 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-updates/universe amd64 Packages \[524 kB\]

Hit [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty Release

Get:9 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-security/main Sources \[164 kB\]

Get:10 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-security/restricted Sources \[5066 B\]

Get:11 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-security/universe Sources \[62.6 kB\]

Get:12 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-security/main amd64 Packages \[758 kB\]

Get:13 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-security/restricted amd64 Packages \[17.8 kB\]

Get:14 [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty-security/universe amd64 Packages \[203 kB\]

Hit [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty/main Sources

Hit [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty/restricted Sources

Hit [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty/universe Sources

Hit [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty/main amd64 Packages

Hit [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty/restricted amd64 Packages

Hit [http://archive.ubuntu.com](http://archive.ubuntu.com) trusty/universe amd64 Packages

Fetched 3836 kB in 3min 18s \(19.3 kB/s\)

Reading package lists...

Reading package lists...

Building dependency tree...

Reading state information...

The following extra packages will be installed:

fortune-mod fortunes-min librecode0

Suggested packages:

x11-utils bsdmainutils

The following NEW packages will be installed:

fortune-mod fortunes fortunes-min librecode0

0 upgraded, 4 newly installed, 0 to remove and 94 not upgraded.

Need to get 1961 kB of archives.

After this operation, 4817 kB of additional disk space will be used.

Get:1 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/main librecode0 amd64 3.6-21 \[771 kB\]

Get:2 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/universe fortune-mod amd64 1:1.99.1-7 \[39.5 kB\]

Get:3 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/universe fortunes-min all 1:1.99.1-7 \[61.8 kB\]

Get:4 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/universe fortunes all 1:1.99.1-7 \[1089 kB\]

debconf: unable to initialize frontend: Dialog

debconf: \(TERM is not set, so the dialog frontend is not usable.\)

debconf: falling back to frontend: Readline

debconf: unable to initialize frontend: Readline

debconf: \(This frontend requires a controlling tty.\)

debconf: falling back to frontend: Teletype

dpkg-preconfigure: unable to re-open stdin:

Fetched 1961 kB in 1min 9s \(28.4 kB/s\)

Selecting previously unselected package librecode0:amd64.

\(Reading database ... 13116 files and directories currently installed.\)

Preparing to unpack .../librecode0\_3.6-21\_amd64.deb ...

Unpacking librecode0:amd64 \(3.6-21\) ...

Selecting previously unselected package fortune-mod.

Preparing to unpack .../fortune-mod\_1%3a1.99.1-7\_amd64.deb ...

Unpacking fortune-mod \(1:1.99.1-7\) ...

Selecting previously unselected package fortunes-min.

Preparing to unpack .../fortunes-min\_1%3a1.99.1-7\_all.deb ...

Unpacking fortunes-min \(1:1.99.1-7\) ...

Selecting previously unselected package fortunes.

Preparing to unpack .../fortunes\_1%3a1.99.1-7\_all.deb ...

Unpacking fortunes \(1:1.99.1-7\) ...

Setting up librecode0:amd64 \(3.6-21\) ...

Setting up fortune-mod \(1:1.99.1-7\) ...

Setting up fortunes-min \(1:1.99.1-7\) ...

Setting up fortunes \(1:1.99.1-7\) ...

Processing triggers for libc-bin \(2.19-0ubuntu6.6\) ...

---&gt; ea034feded71

Removing intermediate container 2a0fb37454de

Step 3/3 : CMD /usr/games/fortune -a \| cowsay

---&gt; Running in 7226f8aef2c5

---&gt; ca5555513a01

Removing intermediate container 7226f8aef2c5

Successfully built ca5555513a01

localhost:docker didi$

localhost:docker didi$

localhost:docker didi$

localhost:docker didi$ docker tag 8e15421920b1 xiazemin/docker-whale:latest

Error response from daemon: no such id: 8e15421920b1

localhost:docker didi$ docker tag ca5555513a01 xiazemin/docker-whale:latest

localhost:docker didi$ docker push xiazemin/docker-whale

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

