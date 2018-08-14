Docker容器运行的时候默认会自动分配一个默认网桥所在网段的IP地址。但很多时候我们可能需要让容器运行在预先指定的静态IP地址上，因为早期的版本不支持静态IP，因此网上大部分方法都是借助pipework等去实现，然而在最新的版本中，Docker已经内嵌支持在启动时指定静态IP了。

Docker守护进程启动以后会创建默认网桥docker0，其IP网段通常为172.17.0.1。在启动Container的时候，Docker将从这个网段自动分配一个IP地址作为容器的IP地址。最新版\(1.10.3\)的Docker内嵌支持在启动容器的时候为其指定静态的IP地址。这里分为三步：

第一步：安装最新版的Docker

备注：操作系统自带的docker的版本太低，不支持静态IP，因此需要自定义安装。

root@localhost:~\# apt-get update

root@localhost:~\# apt-get install curl

root@localhost:~\# curl -fsSL [https://get.docker.com/](https://get.docker.com/) \| sh

root@localhost:~\# docker -v

Docker version 1.10.3, build 20f81dd

第二步：创建自定义网络

备注：这里选取了172.18.0.0网段，也可以指定其他任意空闲的网段

docker network create --subnet=172.18.0.0/16 shadownet

注：shadown为自定义网桥的名字，可自己任意取名。

第三步：在你自定义的网段选取任意IP地址作为你要启动的container的静态IP地址

备注：这里在第二步中创建的网段中选取了172.18.0.10作为静态IP地址。这里以启动shadowsocks为例。

docker run -d -p 2001:2001 --net shadownet --ip 172.18.0.10 oddrationale/docker-shadowsocks -s 0.0.0.0 -p 2001 -k 123456 -m aes-256-cfb

其他

备注1：这里是固定IP地址的一个应用场景的延续，仅作记录用，可忽略不看。

备注2：如果需要将指定IP地址的容器出去的请求的源地址改为宿主机上的其他可路由IP地址，可用iptables来实现。比如将静态IP地址172.18.0.10出去的请求的源地址改成公网IP104.232.36.109\(前提是本机存在这个IP地址\)，可执行如下命令：

iptables -t nat -I POSTROUTING -o eth0 -d  0.0.0.0/0 -s 172.18.0.10  -j SNAT --to-source 104.232.36.109

需要使用Docker虚拟化Hadoop/Spark等测试环境，并且要可以对外提供服务，要求是完全分布式的部署（尽量模拟生产环境）。那么我们会遇到几个问题：



Container IP 是动态分配的



Container IP 是内部IP，外部无法访问（如对外提供HDFS服务可能会遇到Client无法访问DataNode，因为DataNode注册的是内部IP）



针对第一个问题有不少的方案，可以指定静态的IP，对第二个问题，我们可以使用--net=host解决，但这会导致对外只有一个IP，集群各个Slave的端口都要修改。至于pipework简单地看了下，好像也解决不了。



所以目前看上去只能使用看上去不是很优雅的方案解决:\*为Docker宿主网卡绑定多个IP，把这些IP分配给不同的容器。



