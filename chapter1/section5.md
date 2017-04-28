# section5

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