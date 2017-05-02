# Docker管理工具

Docker管理工具Web UI：DockerUI & Shipyard

本文主要介绍两款Docker Web管理工具：DockerUI及Shipyard，并对它们的部署、功能及使用进行对比。

后续会介绍Docker近日最新发布的容器管理利器：swarm。

部署方面

DockerUI

Run cmd docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock dockerui/dockerui

Open your browser to [http://&lt;dockerd](http://<dockerd) host ip&gt;:9000

Shipyard

Run cmd docker run --rm -v /var/run/docker.sock:/var/run/docker.sock shipyard/deploy start

Open your browser to [http://&lt;dockerd](http://<dockerd) host ip&gt;:8080, username: admin, password: shipyard

DockerUI部署很顺利，没遇到任何问题。

Shipyard实际使用过程中遇到一些问题，如：iptables问题。

功能及使用体验方面

两者各有优缺点，比较适合配合使用。

DockerUI

DockerUI基于Docker API，提供等同Docker命令行的大部分功能，支持container管理，image管理。

优点：

支持container批量操作；

支持image管理（虽然比较薄弱）

缺点：

不支持多主机。

Shipyard

Shipyard也是完全基于Docker API，支持container管理、engine管理（一个engine就是监听tcp端口的docker daemon）。

优点：

支持多主机；

支持container及engine资源限制及图形展示；

支持container实例横向扩展；

支持批量创建；

支持创建时自动调度。

缺点：

不支持image管理；

不支持container批量操作。

docker pull hub.c.163.com/longjuxu/shipyard/shipyard:latest

$  docker run --rm -v /var/run/docker.sock:/var/run/docker.sock hub.c.163.com/longjuxu/shipyard/shipyard server

time="2017-05-02T10:53:07Z" level=info msg="shipyard version 3.1.0"

time="2017-05-02T10:53:13Z" level=warning msg="Error creating connection: gorethink: dial tcp: lookup rethinkdb on 192.168.65.1:53: server misbehaving"

time="2017-05-02T10:53:13Z" level=fatal msg="no connections were made when creating the session"

$ docker pull hub.c.163.com/lixiaoming/rethinkdb:latest

latest: Pulling from lixiaoming/rethinkdb

bbe5368a0432: Pull complete

98ff17f2ae39: Pull complete

b2c6d7e5c802: Pull complete

577d9278897e: Pull complete

a3ed95caeb02: Pull complete

c511d384be8e: Pull complete

581e1a19cdf3: Pull complete

Digest: sha256:01b49fb895a162d1311bff7a63916af2932c89ebcb48ce8daa051021dfdf383a

Status: Downloaded newer image for hub.c.163.com/lixiaoming/rethinkdb:latest

$ docker run -it -d --name shipyard-rethinkdb-data --entrypoint /bin/bash hub.c.163.com/lixiaoming/rethinkdb -l

f6c7a7e71886eb512c526d22bf09e8468ce61e903c6ab75d8858045de0846e1a

$ docker run -it -P -d --name shipyard-rethinkdb --volumes-from shipyard-rethinkdb-data  hub.c.163.com/lixiaoming/rethinkdb

6ef526a30ef53817588099f3af88f64d9084ccd8f63b120f620624dd2cefa50a

$ docker run -it -p 8080:8080 -d --name shipyard --link shipyard-rethinkdb:rethinkdb hub.c.163.com/longjuxu/shipyard/shipyard

b2cc621c53ec593e01c5b9076c5519778ca3f17e64929b763e6a1380836e6c7b

docker: Error response from daemon: driver failed programming external connectivity on endpoint shipyard \(47fa1b0042f2857d9873117ed82fdb405db8a3b5d45d73ca2e2a5cb80bed4a57\): Error starting userland proxy: Bind for 0.0.0.0:8080 failed: port is already allocated.

$ docker run -it -p 8089:8089 -d --name shipyard --link shipyard-rethinkdb:rethinkdb hub.c.163.com/longjuxu/shipyard/shipyard

docker: Error response from daemon: Conflict. The container name "/shipyard" is already in use by container b2cc621c53ec593e01c5b9076c5519778ca3f17e64929b763e6a1380836e6c7b. You have to remove \(or rename\) that container to be able to reuse that name..

See 'docker run --help'.

$ docker run -it -p 8089:8089 -d --name shipyard1 --link shipyard-rethinkdb:rethinkdb hub.c.163.com/longjuxu/shipyard/shipyard

d1a269ed556d8a25b7cb78cc3284775940fc9fb84cfaa520bc41fbc3a0786b20

创建容器



$docker run -ti -p 40001:40001 -p 40002:40002 -v /home/docker\_data/rethinkdb:/data --name rethinkdb hub.c.163.com/lixiaoming/rethinkdb sh /etc/rc.local



配置自启动



$vim /etc/rc.local



在"\# Add Start bash under me!"行下添加如下内容:



\#RethinkDB



rethinkdb -d /data --driver-port 40001 --http-port 40002 --bind all --daemon



按“:x”，回车



$exit



启动容器



$docker start rethinkdb



访问http://localhost:40002/



看看能否打开管理页面。

