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

IE访问：http://192.168.220.123:58081/

登录：admin/Shipyard

