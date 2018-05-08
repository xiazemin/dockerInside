### 不同主机间容器通信 {#articleHeader32}

~\# apt-get install iputils-arping bridge-utils -y

Reading package lists... Done

~\# brctl show

bridge name    bridge id        STP enabled    interfaces

～$docker run -d -p 222:22  --net=bridge ifconfig/curl  /usr/sbin/sshd -D

e4ae8449047fb12991e12fd85932ae24e0a9a2eab9a46c50852661900b2d824a

默认也是桥接模式

列出所有docker网络

～$docker network ls

NETWORK ID          NAME                DRIVER              SCOPE

77f42967f231        bridge              bridge              local

8edecb0e4997        host                host                local

e19d371e8b77        none                null                local

要检查其属性，运行

～$ docker network inspect bridge

可以使用docker network create命令并指定选项--driver bridge创建自己的网络，例如

docker network create --driver bridge --subnet 192.168.100.0/24 --ip-range 192.168.100.0/ 24 my-bridge-network创建另一个网桥网络，名称为“my-bridge-network”，子网为192.168.100.0/24。

～／$docker network create --driver bridge --subnet 172.17.100.0/16 --ip-range 172.17.100.0/16  my-bridge-network-docker

Error response from daemon: cannot create network 0861abfb49eb6351be1cbe41e9c1d923bc0359705a5c74f2ad29a584231db07a \(br-0861abfb49eb\): conflicts with network 77f42967f231d0ccc4806c41d348fb221b0fb597efb6dda0680393c877a1edb9 \(docker0\): networks have overlapping IPv4

docker创建的每个网桥网络由docker主机上的网桥接口呈现。、 默认桥网络“bridge”通常具有与其相关联的接口docker0，并且使用docker network create命令创建的每个后续网桥网络将具有与其相关联的新接口。

～／$docker network create docker0

34033bd991144573177dba986df772979f73d578caf8a1c2eb8b2f9f229df272

root@e4ae8449047f:~\#  brctl show docker0

bridge name	bridge id		STP enabled	interfaces

docker0		can't get info No such device



