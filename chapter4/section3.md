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

hub.c.163.com/library/alpine                   latest              4a415e366388        2 months ago        3.99 MB

hub.c.163.com/lixiaoming/rethinkdb             latest              bdc5f86ddaad        2 months ago        442 MB

hub.c.163.com/library/swarm                    latest              36b1e23becab        3 months ago        15.9 MB

hub.c.163.com/longjuxu/shipyard/shipyard       latest              36fb3dc0907d        6 months ago        58.8 MB

hub.c.163.com/longjuxu/shipyard/docker-proxy   latest              cfee14e5d6f2        16 months ago       9.47 MB

hub.c.163.com/longjuxu/microbox/etcd           latest              6aef84b9ec5a        21 months ago       17.9 MB

$ docker run -ti -d --restart=always --name shipyard-rethinkdb hub.c.163.com/lixiaoming/rethinkdb

ac40b635157cdc47f925e50829059656e2368e72a1ee1759d561ae7e157b2527

$ docker run -ti -d -p 54001:4001 -p 57001:7001 --restart=always --name shipyard-discovery  hub.c.163.com/longjuxu/microbox/etcd  -name discovery

15dc24c15a918903aefef2772e00ff7544f2bb8330b556088e05812b1b036470

$ docker run -ti -d -p 2375:2375 --hostname=192.168.220.123 --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 hub.c.163.com/longjuxu/shipyard/docker-proxy

6c4fbc64f0b8875b1d230de03902fe1ae6ff881a03d87751d8e76f9d97563b0c

```
\* \[section7\]\(chapter1/section7.md\)
```

$ docker run -ti -d --restart=always --name shipyard-swarm-manager hub.c.163.com/library/swarm  manage --host tcp://0.0.0.0:3375 etcd://192.168.220.123:54001

dabb77a9f7379c7922d429786eee5f2a89dfce085a6d92030b069fff6863981d

$ docker run -ti -d --restart=always --name shipyard-swarm-agent  hub.c.163.com/library/swarm  join --addr 192.168.220.123:2375 etcd://192.168.220.123:54001

1528e4bb0aea7d02fd7a3361b5274445a0619f54012ec1a3cb89fc0822b30c98

$ docker run -ti -d --restart=always --name shipyard-controller --link shipyard-rethinkdb:rethinkdb --link shipyard-swarm-manager:swarm  -p 58081:8080 hub.c.163.com/longjuxu/shipyard/shipyard server -d tcp://swarm:3375

cfac6462411f82322d07c107c4fdd7355c98b639b4b9190c5cebf18be0052111

$ docker run -ti -d -p 2375:2375 --hostname=192.168.220.127 --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 hub.c.163.com/longjuxu/shipyard/docker-proxy

docker: Error response from daemon: Conflict. The container name "/shipyard-proxy" is already in use by container 6c4fbc64f0b8875b1d230de03902fe1ae6ff881a03d87751d8e76f9d97563b0c. You have to remove \(or rename\) that container to be able to reuse that name..

See 'docker run --help'.

$ docker run -ti -d -p 2375:2375 --hostname=192.168.220.127 --restart=always --name shipyard-proxy1 -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 hub.c.163.com/longjuxu/shipyard/docker-proxy

30c8079ec762af62494822652e2dbd72082b759c11ffe002b858dbfc8fb3d43e

docker: Error response from daemon: driver failed programming external connectivity on endpoint shipyard-proxy1 \(8513a49e0a08a344a49de215b7c106c74dcf469f4549e78fc835a948f2f74713\): Bind for 0.0.0.0:2375 failed: port is already allocated.

