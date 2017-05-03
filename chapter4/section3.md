# Shipyard详解

![](/assets/importshipyard.png)一：proxy从/var/run/docker.sock这个unixsocket获取数据，并被动等待swarm-agent查询

二：swarm-agent通过proxy获取数据，并向etcd推送

以上两个 装在需要被管理的服务器上

三：etcd被动等待swarm-agent推送Docker主机的注册信息

四：swarm-manager使用etcd（shipyard-discovery）获取基本数据

五：shipyard 跟rethinkdb和swarm-manager进行通讯

官方提供的安装命令

\#123服务器

docker run -ti -d --restart=always --name shipyard-rethinkdb rethinkdb

docker run -ti -d -p 54001:4001 -p 57001:7001 --restart=always --name shipyard-discovery  microbox/etcd -name discovery

docker run -ti -d -p 2375:2375 --hostname=192.168.220.123 --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 shipyard/docker-proxy:latest

docker run -ti -d --restart=always --name shipyard-swarm-manager swarm:latest manage --host tcp://0.0.0.0:3375 etcd://192.168.220.123:54001

docker run -ti -d --restart=always --name shipyard-swarm-agent swarm:latest join --addr 192.168.220.123:2375 etcd://192.168.220.123:54001

docker run -ti -d --restart=always --name shipyard-controller --link shipyard-rethinkdb:rethinkdb --link shipyard-swarm-manager:swarm  -p 58081:8080 shipyard/shipyard:latest server -d tcp://swarm:3375

\#127服务器，shipyard有两个节点 一个是自己本身，一个是127

docker run -ti -d -p 2375:2375 --hostname=192.168.220.127 --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 shipyard/docker-proxy:latest

docker run -ti -d --restart=always --name shipyard-swarm-agent swarm:latest join --addr 192.168.220.127:2375 etcd://192.168.220.123:54001

IE访问：[http://192.168.220.123:58081/](http://192.168.220.123:58081/)

登录：admin/Shipyard

－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－

$ docker images

REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE

docker-whale                                   latest              ca5555513a01        4 days ago          277 MB

xiazemin/docker-whale                          latest              ca5555513a01        4 days ago          277 MB

&lt;none&gt;                                         &lt;none&gt;              26371304c9de        4 days ago          188 MB

hub.c.163.com/library/nginx                    latest              46102226f2fd        7 days ago          109 MB

nginx                                          latest              46102226f2fd        7 days ago          109 MB

php                                            7.0-cli             1f1e1ccbb091        8 days ago          364 MB

hub.c.163.com/library/php                      latest              b4326367db92        2 weeks ago         365 MB

ubuntu                                         14.04               302fa07d8117        2 weeks ago         188 MB

hub.c.163.com/library/mysql                    latest              d5127813070b        3 weeks ago         407 MB

alpine                                         latest              4a415e366388        2 months ago        3.99 MB

hub.c.163.com/library/alpine                   latest              4a415e366388        2 months ago        3.99 MB

hub.c.163.com/lixiaoming/rethinkdb             latest              bdc5f86ddaad        2 months ago        442 MB

hub.c.163.com/library/swarm                    latest              36b1e23becab        3 months ago        15.9 MB

hub.c.163.com/yswtrue/php-fpm                  latest              7627aeff9ce5        3 months ago        408 MB

hub.c.163.com/ncetest001/nosshd                run5                d3bf7fab9ddd        4 months ago        227 MB

hub.c.163.com/liuinstein/sshd                  latest              ebdbd0de15bb        4 months ago        223 MB

hub.c.163.com/longjuxu/shipyard/shipyard       latest              36fb3dc0907d        6 months ago        58.8 MB

hub.c.163.com/public/redis                     2.8.4               4888527e1254        7 months ago        190 MB

hub.c.163.com/muicoder/ambassador              latest              e3e5c70243f4        10 months ago       8.01 MB

hub.c.163.com/longjuxu/shipyard/docker-proxy   latest              cfee14e5d6f2        16 months ago       9.47 MB

hub.c.163.com/longjuxu/microbox/etcd           latest              6aef84b9ec5a        21 months ago       17.9 MB

docker/whalesay                                latest              6b362a9f73eb        23 months ago       247 MB

tanghuimin0713/ubuntu\_amd64                    14.04               9772288ea0c5        2 years ago         222 MB

localhost:dockerui didi$ docker run -ti -d --restart=always --name shipyard-rethinkdb hub.c.163.com/lixiaoming/rethinkdb

docker: Error response from daemon: Conflict. The container name "/shipyard-rethinkdb" is already in use by container 6ef526a30ef53817588099f3af88f64d9084ccd8f63b120f620624dd2cefa50a. You have to remove \(or rename\) that container to be able to reuse that name..

See 'docker run --help'.

localhost:dockerui didi$ docker rm shipyard-rethinkdb

shipyard-rethinkdb

localhost:dockerui didi$ docker run -ti -d --restart=always --name shipyard-rethinkdb hub.c.163.com/lixiaoming/rethinkdb

ac40b635157cdc47f925e50829059656e2368e72a1ee1759d561ae7e157b2527

localhost:dockerui didi$ docker run -ti -d -p 54001:4001 -p 57001:7001 --restart=always --name shipyard-discovery  hub.c.163.com/longjuxu/microbox/etcd  -name discovery

15dc24c15a918903aefef2772e00ff7544f2bb8330b556088e05812b1b036470

localhost:dockerui didi$ docker run -ti -d -p 2375:2375 --hostname=192.168.220.123 --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 hub.c.163.com/longjuxu/shipyard/docker-proxy

6c4fbc64f0b8875b1d230de03902fe1ae6ff881a03d87751d8e76f9d97563b0c

    \* \[section7\]\(chapter1/section7.md\)

localhost:dockerui didi$ docker run -ti -d --restart=always --name shipyard-swarm-manager hub.c.163.com/library/swarm  manage --host tcp://0.0.0.0:3375 etcd://192.168.220.123:54001

dabb77a9f7379c7922d429786eee5f2a89dfce085a6d92030b069fff6863981d

localhost:dockerui didi$ docker run -ti -d --restart=always --name shipyard-swarm-agent  hub.c.163.com/library/swarm  join --addr 192.168.220.123:2375 etcd://192.168.220.123:54001

1528e4bb0aea7d02fd7a3361b5274445a0619f54012ec1a3cb89fc0822b30c98

localhost:dockerui didi$ docker run -ti -d --restart=always --name shipyard-controller --link shipyard-rethinkdb:rethinkdb --link shipyard-swarm-manager:swarm  -p 58081:8080 hub.c.163.com/longjuxu/shipyard/shipyard server -d tcp://swarm:3375

cfac6462411f82322d07c107c4fdd7355c98b639b4b9190c5cebf18be0052111

localhost:dockerui didi$ docker run -ti -d -p 2375:2375 --hostname=192.168.220.127 --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 hub.c.163.com/longjuxu/shipyard/docker-proxy

docker: Error response from daemon: Conflict. The container name "/shipyard-proxy" is already in use by container 6c4fbc64f0b8875b1d230de03902fe1ae6ff881a03d87751d8e76f9d97563b0c. You have to remove \(or rename\) that container to be able to reuse that name..

See 'docker run --help'.

localhost:dockerui didi$ docker run -ti -d -p 2375:2375 --hostname=192.168.220.127 --restart=always --name shipyard-proxy1 -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 hub.c.163.com/longjuxu/shipyard/docker-proxy

30c8079ec762af62494822652e2dbd72082b759c11ffe002b858dbfc8fb3d43e

docker: Error response from daemon: driver failed programming external connectivity on endpoint shipyard-proxy1 \(8513a49e0a08a344a49de215b7c106c74dcf469f4549e78fc835a948f2f74713\): Bind for 0.0.0.0:2375 failed: port is already allocated.

