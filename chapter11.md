# 容器间网络通信设置(Pipework和Open vSwitch)

自从Docker容器出现以来，容器的网络通信就一直是被关注的焦点，也是生产环境的迫切需求。容器的网络通信又可以分为两大方面：单主机容器上的相互通信，和跨主机的容器相互通信。下面将分别针对这两方面，对容器的通信原理进行简单的分析，帮助大家更好地使用docker。前面已经在Docker容器学习梳理--基础知识（2）这一篇中详细介绍了Docker的网络配置以及pipework工具。
　docker单主机容器通信
基于对net namespace的控制，docker可以为在容器创建隔离的网络环境，在隔离的网络环境下，容器具有完全独立的网络栈，与宿主机隔离，也可以使容器共享主机或者其他容器的网络命名空间，基本可以满足开发者在各种场景下的需要。按docker官方的说法，docker容器的网络有五种模式：
1）bridge模式，--net=bridge\(默认\)
这是dokcer网络的默认设置，为容器创建独立的网络命名空间，容器具有独立的网卡等所有单独的网络栈，是最常用的使用方式。
在docker run启动容器的时候，如果不加--net参数，就默认采用这种网络模式。安装完docker，系统会自动添加一个供docker使用的网桥docker0，我们创建一个新的容器时，容器通过DHCP获取一个与docker0同网段的IP地址，并默认连接到docker0网桥，以此实现容器与宿主机的网络互通。
2）host模式，--net=host
这个模式下创建出来的容器，直接使用容器宿主机的网络命名空间。将不拥有自己独立的Network Namespace，即没有独立的网络环境。它使用宿主机的ip和端口。
3）none模式，--net=none
为容器创建独立网络命名空间，但不为它做任何网络配置，容器中只有lo，用户可以在此基础上，对容器网络做任意定制。
这个模式下，dokcer不为容器进行任何网络配置。需要我们自己为容器添加网卡，配置IP。

因此，若想使用pipework配置docker容器的ip地址，必须要在none模式下才可以。

4）其他容器模式（即container模式），--net=container:NAME\_or\_ID

与host模式类似，只是容器将与指定的容器共享网络命名空间。

这个模式就是指定一个已有的容器，共享该容器的IP和端口。除了网络方面两个容器共享，其他的如文件系统，进程等还是隔离开的。

5）用户自定义：docker 1.9版本以后新增的特性，允许容器使用第三方的网络实现或者创建单独的bridge网络，提供网络隔离能力。
这些网络模式在相互网络通信方面的对比如下所示：
南北向通信指容器与宿主机外界的访问机制，东西向流量指同一宿主机上，与其他容器相互访问的机制。
host模
由于容器和宿主机共享同一个网络命名空间，换言之，容器的IP地址即为宿主机的IP地址。所以容器可以和宿主机一样，使用宿主机的任意网卡，实现和外界的通信。采用host模式的容器，可以直接使用宿主机的IP地址与外界进行通信，若宿主机具有公有IP，那么容器也拥有这个公有IP。同时容器内服务的端口也可以使用宿主机的端口，

无需额外进行NAT转换，而且由于容器通信时，不再需要通过linuxbridge等方式转发或者数据包的拆封，性能上有很大优势。

当然，这种模式有优势，也就有劣势，主要包括以下几个方面：

1）最明显的就是容器不再拥有隔离、独立的网络栈。容器会与宿主机竞争网络栈的使用，并且容器的崩溃就可能导致宿主机崩溃，在生产环境中，这种问题可能是不被允许的。

2）容器内部将不再拥有所有的端口资源，因为一些端口已经被宿主机服务、bridge模式的容器端口绑定等其他服务占用掉了。



　　bridge模式







bridge模式是docker默认的，也是开发者最常使用的网络模式。在这种模式下，docker为容器创建独立的网络栈，保证容器内的进程使用独立的网络环境，

实现容器之间、容器与宿主机之间的网络栈隔离。同时，通过宿主机上的docker0网桥，容器可以与宿主机乃至外界进行网络通信。

其网络模型可以参考下图： 



 









从上面的网络模型可以看出，容器从原理上是可以与宿主机乃至外界的其他机器通信的。同一宿主机上，容器之间都是连接掉docker0这个网桥上的，它可以作为虚拟交换机使容器可以相互通信。

然而，由于宿主机的IP地址与容器veth pair的 IP地址均不在同一个网段，故仅仅依靠veth pair和namespace的技术，还不足以使宿主机以外的网络主动发现容器的存在。为了使外界可以方位容器中的进程，docker采用了端口绑定的方式，也就是通过iptables的NAT，将宿主机上的端口

端口流量转发到容器内的端口上。

举一个简单的例子，使用下面的命令创建容器，并将宿主机的3306端口绑定到容器的3306端口： 

docker run -tid --name db -p 3306:3306 MySQL

在宿主机上，可以通过iptables -t nat -L -n，查到一条DNAT规则：

DNAT tcp -- 0.0.0.0/0 0.0.0.0/0 tcp dpt:3306 to:172.17.0.5:3306

上面的172.17.0.5即为bridge模式下，创建的容器IP。

很明显，bridge模式的容器与外界通信时，必定会占用宿主机上的端口，从而与宿主机竞争端口资源，对宿主机端口的管理会是一个比较大的问题。同时，由于容器与外界通信是基于三层上iptables NAT，性能和效率上的损耗是可以预见的。



　　none模式







在这种模式下，容器有独立的网络栈，但不包含任何网络配置，只具有lo这个loopback网卡用于进程通信。也就是说，none模式为容器做了最少的网络设置，

但是俗话说得好“少即是多”，在没有网络配置的情况下，通过第三方工具或者手工的方式，开发这任意定制容器的网络，提供了最高的灵活性

　　其他容器（container）模式







其他网络模式是docker中一种较为特别的网络的模式。在这个模式下的容器，会使用其他容器的网络命名空间，其网络隔离性会处于bridge桥接模式与host模式之间。

当容器共享其他容器的网络命名空间，则在这两个容器之间不存在网络隔离，而她们又与宿主机以及除此之外其他的容器存在网络隔离。其网络模型可以参考下图： 

 









在这种模式下的容器可以通过localhost来同一网络命名空间下的其他容器，传输效率较高。而且这种模式还节约了一定数量的网络资源，但它并没有改变容器与外界通信的方式。

在一些特殊的场景中非常有用，例如，kubernetes的pod，kubernetes为pod创建一个基础设施容器，同一pod下的其他容器都以其他容器模式共享这个基础设施容器的网络命名空间，

相互之间以localhost访问，构成一个统一的整体。



　　用户定义网络模式







在用户定义网络模式下，开发者可以使用任何docker支持的第三方网络driver来定制容器的网络。并且，docker 1.9以上的版本默认自带了bridge和overlay两种类型的自定义网络driver。可以用于集成calico、weave、openvswitch等第三方厂商的网络实现。 

除了docker自带的bridge driver，其他的几种driver都可以实现容器的跨主机通信。而基于bdrige driver的网络，docker会自动为其创建iptables规则，

保证与其他网络之间、与docker0之间的网络隔离。 

例如，使用下面的命令创建一个基于bridge driver的自定义网络：

docker network create bri1

则docker会自动生成如下的iptables规则，保证不同网络上的容器无法互相通信。

-A DOCKER-ISOLATION -i br-8dba6df70456 -o docker0 -j DROP 

-A DOCKER-ISOLATION -i docker0 -o br-8dba6df70456 -j DROP

除此之外，bridge driver的所有行为都和默认的bridge模式完全一致。而overlay及其他driver，则可以实现容器的跨主机通信。

　　docker跨主机容器通信







早期大家的跨主机通信方案主要有以下几种：

1）容器使用host模式：容器直接使用宿主机的网络，这样天生就可以支持跨主机通信。虽然可以解决跨主机通信问题，但这种方式应用场景很有限，容易出现端口冲突，也无法做到隔离网络环境，

一个容器崩溃很可能引起整个宿主机的崩溃。

2）端口绑定：通过绑定容器端口到宿主机端口，跨主机通信时，使用主机IP+端口的方式访问容器中的服务。显而易见，这种方式仅能支持网络栈的四层及以上的应用，并且容器与宿主机紧耦合，

很难灵活的处理，可扩展性不佳。

3）docker外定制容器网络：在容器通过docker创建完成后，然后再通过修改容器的网络命名空间来定义容器网络。典型的就是很久以前的pipework，容器以none模式创建，pipework通过进入容器

的网络命名空间为容器重新配置网络，这样容器网络可以是静态IP、vxlan网络等各种方式，非常灵活，容器启动的一段时间内会没有IP，明显无法在大规模场景下使用，只能在实验室中测试使用。

4）第三方SDN定义容器网络：使用Open vSwitch或Flannel等第三方SDN工具，为容器构建可以跨主机通信的网络环境。这些方案一般要求各个主机上的docker0网桥的cidr不同，以避免出现IP冲突

的问题，限制了容器在宿主机上的可获取IP范围。并且在容器需要对集群外提供服务时，需要比较复杂的配置，对部署实施人员的网络技能要求比较高。



上面这些方案有各种各样的缺陷，同时也因为跨主机通信的迫切需求，docker 1.9版本时，官方提出了基于vxlan的overlay网络实现，原生支持容器的跨主机通信。同时，还支持通过libnetwork的

plugin机制扩展各种第三方实现，从而以不同的方式实现跨主机通信。就目前社区比较流行的方案来说，跨主机通信的基本实现方案有以下几种：

1）基于隧道的overlay网络：按隧道类型来说，不同的公司或者组织有不同的实现方案。docker原生的overlay网络就是基于vxlan隧道实现的。ovn则需要通过geneve或者stt隧道来实现的。flannel

最新版本也开始默认基于vxlan实现overlay网络。

2）基于包封装的overlay网络：基于UDP封装等数据包包装方式，在docker集群上实现跨主机网络。典型实现方案有weave、flannel的早期版本。

3）基于三层实现SDN网络：基于三层协议和路由，直接在三层上实现跨主机网络，并且通过iptables实现网络的安全隔离。典型的方案为Project Calico。同时对不支持三层路由的环境，Project Calico还提供了基于IPIP封装的跨主机网络实现

　　Dokcer通过使用Linux桥接提供容器之间的通信，docker0桥接接口的目的就是方便Docker管理。当Docker daemon启动时需要做以下操作







a）如果docker0不存在则创建

b）搜索一个与当前路由不冲突的ip段

c）在确定的范围中选择 ip

d）绑定ip到 docker0

　　列出当前主机网桥







\[iyunv@localhost ~\]\# brctl show

bridge name    bridge id           STP enabled   interfaces

docker0        8000.02426f15541e   no            vethe833b02

　　查看当前 docker0 ip







\[iyunv@localhost ~\]\# ifconfig

docker0: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 1500

inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0

inet6 fe80::42:6fff:fe15:541e  prefixlen 64  scopeid 0x20&lt;link&gt;

ether 02:42:6f:15:54:1e  txqueuelen 0  \(Ethernet\)

RX packets 120315  bytes 828868638 \(790.4 MiB\)

RX errors 0  dropped 0  overruns 0  frame 0

TX packets 132565  bytes 100884398 \(96.2 MiB\)

TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

................

　　在容器运行时，每个容器都会分配一个特定的虚拟机口并桥接到docker0。每个容器都会配置同docker0 ip相同网段的专用ip 地址，docker0的IP地址被用于所有容器的默认网关。

一般启动的容器中ip默认是172.17.0.1/24网段的。







\[iyunv@linux-node2 ~\]\# docker images

REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE

centos                       latest              67591570dd29        3 months ago        191.8 MB

\[iyunv@linux-node2 ~\]\# docker run -t -i --name my-test centos /bin/bash

\[iyunv@c5217f7bd44c /\]\# 

\[iyunv@linux-node2 ~\]\# docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES

c5217f7bd44c        centos              "/bin/bash"         10 seconds ago      Up 10 seconds                                my-test

\[iyunv@linux-node2 ~\]\# docker inspect c5217f7bd44c\|grep IPAddress

"SecondaryIPAddresses": null,

"IPAddress": "172.17.0.2",

"IPAddress": "172.17.0.2",

　　那么能不能在创建容器的时候指定特定的ip呢？这是当然可以实现的！

　　注意：宿主机的ip路由转发功能一定要打开，否则所创建的容器无法联网！







\[iyunv@localhost ~\]\# cat /proc/sys/net/ipv4/ip\_forward

1

\[iyunv@localhost ~\]\# 

\[iyunv@localhost ~\]\# docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

6e64eade06d1        docker.io/centos    "/bin/bash"         10 seconds ago      Up 9 seconds                            my-centos

\[iyunv@localhost ~\]\# docker run -itd --net=none --name=container1 docker.io/centos

5e5bdbc4d9977e6bcfa40e0a9c3be10806323c9bf5a60569775903d345869b09

\[iyunv@localhost ~\]\# docker attach container1

\[iyunv@5e5bdbc4d997 /\]\# ping www.baidu.com

PING www.a.shifen.com \(61.135.169.121\) 56\(84\) bytes of data.

64 bytes from 61.135.169.121 \(61.135.169.121\): icmp\_seq=1 ttl=53 time=2.09 ms

64 bytes from 61.135.169.121 \(61.135.169.121\): icmp\_seq=2 ttl=53 time=2.09 ms

关闭ip路由转发功能，容器即不能联网

\[iyunv@localhost ~\]\# echo 0 &gt; /proc/sys/net/ipv4/ip\_forward

\[iyunv@localhost ~\]\# cat /proc/sys/net/ipv4/ip\_forward

0

\[iyunv@5e5bdbc4d997 /\]\# ping www.baidu.com        //ping不通~

　　一、创建容器使用特定范围的IP







Docker 会尝试寻找没有被主机使用的ip段，尽管它适用于大多数情况下，但是它不是万能的，有时候我们还是需要对ip进一步规划。

Docker允许你管理docker0桥接或者通过-b选项自定义桥接网卡，需要安装bridge-utils软件包。操作流程如下：

a）确保docker的进程是停止的

b）创建自定义网桥

c）给网桥分配特定的ip

d）以-b的方式指定网桥



具体操作过程如下（比如创建容器的时候，指定ip为192.168.5.1/24网段的）：

\[iyunv@localhost ~\]\# service docker stop

\[iyunv@localhost ~\]\# ip link set dev docker0 down

\[iyunv@localhost ~\]\# brctl delbr docker0

\[iyunv@localhost ~\]\# brctl addbr bridge0

\[iyunv@localhost ~\]\# ip addr add 192.168.5.1/24 dev bridge0      //注意，这个192.168.5.1就是所建容器的网关地址。通过docker inspect container\_id能查看到

\[iyunv@localhost ~\]\# ip link set dev bridge0 up

\[iyunv@localhost ~\]\# ip addr show bridge0

\[iyunv@localhost ~\]\# vim /etc/sysconfig/docker      //即将虚拟的桥接口由默认的docker0改为bridge0

将

OPTIONS='--selinux-enabled --log-driver=journald'

改为

OPTIONS='--selinux-enabled --log-driver=journald -b=bridge0'    //即添加-b=bridge0



\[iyunv@localhost ~\]\# service docker restart



--------------------------------------------------------------------------------------

上面是centos7下的操作步骤,下面提供下ubuntu下的操作步骤：

$ sudo service docker stop

$ sudo ip link set dev docker0 down

$ sudo brctl delbr docker0

$ sudo brctl addbr bridge0

$ sudo ip addr add 192.168.5.1/24 dev bridge0

$ sudo ip link set dev bridge0 up

$ ip addr show bridge0

$ echo 'DOCKER\_OPTS="-b=bridge0"' &gt;&gt; /etc/default/docker

$ sudo service docker start

--------------------------------------------------------------------------------------



然后创建容器，查看下容器ip是否为设定的192.168.5.1/24网段的

\[iyunv@localhost ~\]\# docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

docker.io/ubuntu    latest              0ef2e08ed3fa        2 weeks ago         130 MB

centos7             7.3.1611            d5ebea14da54        3 weeks ago         311 MB



\[iyunv@localhost ~\]\# docker run -t -i --name test2 centos7:7.3.1611 /bin/bash

\[iyunv@224facf8e054 /\]\#



\[iyunv@localhost ~\]\# docker run -t -i --name test1 docker.io/ubuntu /bin/bash

root@f5b1bfc2811a:/\#



\[iyunv@localhost ~\]\# docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

224facf8e054        centos7:7.3.1611    "/bin/bash"         46 minutes ago      Up 46 minutes                           test2

f5b1bfc2811a        docker.io/ubuntu    "/bin/bash"         47 minutes ago      Up 5 minutes                            test1

\[iyunv@localhost ~\]\# docker inspect --format='{{.NetworkSettings.IPAddress}}' f5b1bfc2811a

192.168.5.2

\[iyunv@localhost ~\]\# docker inspect --format='{{.NetworkSettings.IPAddress}}' 224facf8e054

192.168.5.3

\[iyunv@localhost ~\]\# brctl show

bridge name   bridge id           STP enabled     interfaces

bridge0       8000.ba141fa20c91   no              vethe7e227b

vethf382771

　　使用pipework给容器设置一个固定的ip







可以利用pipework为容器指定一个固定的ip，操作方法非常简单，如下：

\[iyunv@node1 ~\]\# brctl addbr br0

\[iyunv@node1 ~\]\# ip link set dev br0 up

\[iyunv@node1 ~\]\# ip addr add 192.168.114.1/24 dev br0                        //这个ip相当于br0网桥的网关ip，可以随意设定。

\[iyunv@node1 ~\]\# docker run -ti -d --net=none --name=my-test1 docker.io/nginx /bin/bash

\[iyunv@node1 ~\]\# pipework br0 -i eth0 my-test1 192.168.114.100/24@192.168.114.1

\[iyunv@node1 ~\]\# docker exec -ti my-test1 /bin/bash

root@cf370a090f63:/\# ip addr

1: lo: &lt;LOOPBACK,UP,LOWER\_UP&gt; mtu 65536 qdisc noqueue state UNKNOWN group default 

link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

inet 127.0.0.1/8 scope host lo

valid\_lft forever preferred\_lft forever

inet6 ::1/128 scope host 

valid\_lft forever preferred\_lft forever

57: eth0@if58: &lt;BROADCAST,MULTICAST,UP,LOWER\_UP&gt; mtu 1500 qdisc pfifo\_fast state UP group default qlen 1000

link/ether b2:c1:8d:92:33:e2 brd ff:ff:ff:ff:ff:ff link-netnsid 0

inet 192.168.114.100/24 brd 192.168.114.255 scope global eth0

valid\_lft forever preferred\_lft forever

inet6 fe80::b0c1:8dff:fe92:33e2/64 scope link 

valid\_lft forever preferred\_lft forever



再启动一个容器

\[iyunv@node1 ~\]\# docker run -ti -d --net=none --name=my-test2 docker.io/nginx /bin/bash

\[iyunv@node1 ~\]\# pipework br0 -i eth0 my-test12 192.168.114.200/24@192.168.114.1

\[iyunv@node1 ~\]\# pipework br0 -i eth0 my-test2 192.168.114.200/24@192.168.114.1

这样，my-test1容器和my-test2容器在同一个宿主机上，所以它们固定后的ip是可以相互ping通的，如果是在不同的宿主机上，则就无法ping通！

所以说：

这样使用pipework指定固定ip的容器，在同一个宿主机下的容器间的ip是可以相互ping通的，但是跨主机的容器通过这种方式固定ip后就不能ping通了。

跨主机的容器间的通信可以看下面的介绍。

　　二、不同主机间的容器通信（pipework  config docker container ip）







我的centos7测试机上的docker是yum安装的，默认自带pipework工具，所以就不用在另行安装它了。

-----------------------------------------------------------------------------------------------

如果没有pipework工具，可以安装下面步骤进行安装：

\# git clone https://github.com/jpetazzo/pipework.git

\# sudo cp -rp pipework/pipework /usr/local/bin/



安装相应依赖软件\(网桥\)

\#sudo apt-get install iputils-arping bridge-utils -y

-----------------------------------------------------------------------------------------------



查看Docker宿主机上的桥接网络

\[iyunv@linux-node2 ~\]\# brctl show

bridge name   bridge id           STP enabled   interfaces

docker0       8000.02426f15541e   no            veth92d132f



有两种方式做法：

1）可以选择删除docker0，直接把docker的桥接指定为br0；

2）也可以选择保留使用默认docker0的配置，这样单主机容器之间的通信可以通过docker0；

跨主机不同容器之间通过pipework将容器的网卡桥接到br0上，这样跨主机容器之间就可以通信了。



如果保留了docker0，则容器启动时不加--net=none参数，那么本机容器启动后就是默认的docker0自动分配的ip（默认是172.17.1.0/24网段），它们之间是可以通信的；

跨宿主机的容器创建时要加--net=none参数，待容器启动后通过pipework给容器指定ip，这样跨宿主机的容器ip是在同一网段内的同网段地址，因此可以通信。



一般来说：最好在创建容器的时候加上--net=none，防止自动分配的IP在局域网中有冲突。若是容器创建后自动获取ip，下次容器启动会ip有变化，可能会和物理网段中的ip冲突



---------------------------------------------------------------------------------------------------

实例说明如下：

宿主机信息

ip：192.168.1.23          （网卡设备为eth0）

gateway：192.168.1.1

netmask：255.255.255.0

1）删除虚拟桥接卡docker0的配置

\[iyunv@localhost ~\]\# service docker stop

\[iyunv@localhost ~\]\# ip link set dev docker0 down 

\[iyunv@localhost ~\]\# brctl delbr docker0

\[iyunv@localhost ~\]\# brctl addbr br0

\[iyunv@localhost ~\]\# ip link set dev br0 up       

\[iyunv@localhost ~\]\# ip addr del 192.168.1.23/24 dev eth0       //删除宿主机网卡的IP（如果是使用这个地址进行的远程连接，这一步操作后就会断掉；如果是使用外网地址连接的话，就不会断开）

\[iyunv@localhost ~\]\# ip addr add 192.168.1.23/24 dev br0        //将宿主主机的ip设置到br0

\[iyunv@localhost ~\]\# brctl addif br0 eth0                        //将宿主机网卡挂到br0上

\[iyunv@localhost ~\]\# ip route del default                       //删除默认的原路由，其实就是eth0上使用的原路由192.168.1.1（这步小心，注意删除后要保证机器能远程连接上，最好是通过外网ip远程连的。别删除路由后，远程连接不上，中断了）

\[iyunv@localhost ~\]\# ip route add default via 192.168.1.1 dev br0      //为br0设置路由

\[iyunv@localhost ~\]\# vim /etc/sysconfig/docker                 //即将虚拟的桥接口由默认的docker0改为bridge0

将

OPTIONS='--selinux-enabled --log-driver=journald'

改为

OPTIONS='--selinux-enabled --log-driver=journald -b=br0'    //即添加-b=br0



\[iyunv@localhost ~\]\# service docker start



启动一个手动设置网络的容器

\[iyunv@localhost ~\]\# docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

6e64eade06d1        docker.io/centos    "/bin/bash"         10 seconds ago      Up 9 seconds                            my-centos

\[iyunv@localhost ~\]\# docker run -itd --net=none --name=my-test1 docker.io/centos



为my-test1容器设置一个与桥接物理网络同地址段的ip（如下，"ip@gateway"）

默认不指定网卡设备名，则默认添加为eth0。可以通过-i参数添加网卡设备名

\[iyunv@localhost ~\]\# pipework br0 -i eth0 my-test1 192.168.1.190/24@192.168.1.1



同理，在其他机器上启动容器，并类似上面用pipework设置一个同网段类的ip，这样跨主机的容器就可以相互ping通了！



--------------------------------------------------------------------------------------------------

2）保留默认虚拟桥接卡docker0的配置

\[iyunv@localhost ~\]\# cd /etc/sysconfig/network-scripts/

\[iyunv@localhost network-scripts\]\# cp ifcfg-eth0 ifcfg-eth0.bak

\[iyunv@localhost network-scripts\]\# cp ifcfg-eth0 ifcfg-br0

\[iyunv@localhost network-scripts\]\# vim ifcfg-eth0            //增加BRIDGE=br0，删除IPADDR,NETMASK,GATEWAY,DNS的设置

......

BRIDGE=br0

\[iyunv@localhost network-scripts\]\# vim ifcfg-br0            //修改DEVICE为br0,Type为Bridge,把eth0的网络设置设置到这里来（里面应该有ip，网关，子网掩码或DNS设置）

......

TYPE=Bridge

DEVICE=br0



\[iyunv@localhost network-scripts\]\# service network restart



\[iyunv@localhost network-scripts\]\# service docker restart



开启一个容器并指定网络模式为none（这样，创建的容器就不会通过docker0自动分配ip了，而是根据pipework工具自定ip指定）

\[iyunv@localhost network-scripts\]\# docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

docker.io/centos    latest              67591570dd29        3 months ago        191.8 MB

\[iyunv@localhost network-scripts\]\# docker run -itd --net=none --name=my-centos docker.io/centos /bin/bash

6e64eade06d1eb20be3bd22ece2f79174cd033b59182933f7bbbb502bef9cb0f



接着给容器配置网络

\[iyunv@localhost network-scripts\]\# pipework br0 -i eth0 my-centos 192.168.1.150/24@192.168.1.1

\[iyunv@localhost network-scripts\]\# docker attach 6e64eade06d1

\[iyunv@6e64eade06d1 /\]\# ifconfig eth0                 //若没有ifconfig命令，可以yum安装net-tools工具

eth0      Link encap:Ethernet  HWaddr 86:b6:6b:e8:2e:4d

inet addr:192.168.1.150  Bcast:0.0.0.0  Mask:255.255.255.0

inet6 addr: fe80::84b6:6bff:fee8:2e4d/64 Scope:Link

UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

RX packets:8 errors:0 dropped:0 overruns:0 frame:0

TX packets:9 errors:0 dropped:0 overruns:0 carrier:0

collisions:0 txqueuelen:1000

RX bytes:648 \(648.0 B\)  TX bytes:690 \(690.0 B\)

\[iyunv@6e64eade06d1 /\]\# route -n

Kernel IP routing table

Destination     Gateway         Genmask         Flags Metric Ref    Use Iface

0.0.0.0         192.168.1.1     0.0.0.0         UG    0      0        0 eth0

192.168.115.0   0.0.0.0         255.255.255.0   U     0      0        0 eth0



另外pipework不能添加静态路由，如果有需求则可以在run的时候加上--privileged=true 权限在容器中手动添加，但这种方法安全性有缺陷。

除此之外，可以通过ip netns（--help参考帮助）添加静态路由，以避免创建容器使用--privileged=true选项造成一些不必要的安全问题：



如下获取指定容器的pid

\[iyunv@localhost network-scripts\]\# docker inspect --format="{{ .State.Pid }}" 6e64eade06d1

7852

\[iyunv@localhost network-scripts\]\# ln -s /proc/7852/ns/net /var/run/netns/7852

\[iyunv@localhost network-scripts\]\# ip netns exec 7852 ip route add 192.168.0.0/16 dev eth0 via 192.168.1.1

\[iyunv@localhost network-scripts\]\# ip netns exec 7852 ip route    //添加成功

192.168.0.0/16 via 192.168.1.1 dev eth0



同理，在其它宿主机进行相应的配置，新建容器并使用pipework添加虚拟网卡桥接到br0，如此创建的容器间就可以相互通信了。

　　-----------------------------------------------几个报错出来---------------------------------------------







1）重启网卡报错如下：

\# systemctl restart network

......

Nov 23 22:09:08 hdcoe02 systemd\[1\]: network.service: control process exited, code=exited status=1  

Nov 23 22:09:08 hdcoe02 systemd\[1\]: Failed to start LSB: Bring up/down networking.  

Nov 23 22:09:08 hdcoe02 systemd\[1\]: Unit network.service entered failed state.&lt;/span&gt;  

解决办法：

\# systemctl enable NetworkManager-wait-online.service

\# systemctl stop NetworkManager

\# systemctl  restart network.service

2）创建容器，出现下面告警

WARNING: IPv4 forwarding is disabled. Networking will not work.

解决办法：

\#vim /usr/lib/sysctl.d/00-system.conf

添加如下代码：

net.ipv4.ip\_forward=1

重启network服务

\# systemctl restart network



　　-----------------------------------------------------------------------------------------------------------------------------------

其实除了上面使用的pipework工具还，还可以使用虚拟交换机\(Open vSwitch\)进行docker容器间的网络通信，废话不多说，下面说下Open vSwitch的使用：







一、在Server1和Server2上分别安装open vswitch

\[iyunv@Slave1 ~\]\# \# yum -y install wget openssl-devel kernel-devel

\[iyunv@Slave1 ~\]\# yum groupinstall "Development Tools"

\[iyunv@Slave1 ~\]\# adduser ovswitch

\[iyunv@Slave1 ~\]\# su - ovswitch

\[iyunv@Slave1 ~\]$ wget http://openvswitch.org/releases/openvswitch-2.3.0.tar.gz

\[iyunv@Slave1 ~\]$ tar -zxvpf openvswitch-2.3.0.tar.gz

\[iyunv@Slave1 ~\]$ mkdir -p ~/rpmbuild/SOURCES

\[iyunv@Slave1 ~\]$ sed 's/openvswitch-kmod, //g' openvswitch-2.3.0/rhel/openvswitch.spec &gt; openvswitch-2.3.0/rhel/openvswitch\_no\_kmod.spec

\[iyunv@Slave1 ~\]$ cp openvswitch-2.3.0.tar.gz rpmbuild/SOURCES/



\[iyunv@Slave1 ~\]$ rpmbuild -bb --without check ~/openvswitch-2.3.0/rhel/openvswitch\_no\_kmod.spec



\[iyunv@Slave1 ~\]$ exit



\[iyunv@Slave1 ~\]\# yum localinstall /home/ovswitch/rpmbuild/RPMS/x86\_64/openvswitch-2.3.0-1.x86\_64.rpm

\[iyunv@Slave1 ~\]\# mkdir /etc/openvswitch

\[iyunv@Slave1 ~\]\# setenforce 0

\[iyunv@Slave1 ~\]\# systemctl start openvswitch.service

\[iyunv@Slave1 ~\]\# systemctl  status openvswitch.service -l



二、在Slave1和Slave2上建立OVS Bridge并配置路由

1）在Slave1宿主机上设置docker容器内网ip网段172.17.1.0/24

\[iyunv@Slave1 ~\]\# vim /proc/sys/net/ipv4/ip\_forward

1

\[iyunv@Slave1 ~\]\# ovs-vsctl add-br obr0

\[iyunv@Slave1 ~\]\# ovs-vsctl add-port obr0 gre0 -- set Interface gre0 type=gre options:remote\_ip=192.168.115.5



\[iyunv@Slave1 ~\]\# brctl addbr kbr0

\[iyunv@Slave1 ~\]\# brctl addif kbr0 obr0

\[iyunv@Slave1 ~\]\# ip link set dev docker0 down

\[iyunv@Slave1 ~\]\# ip link del dev docker0



\[iyunv@Slave1 ~\]\# vim /etc/sysconfig/network-scripts/ifcfg-kbr0

ONBOOT=yes

BOOTPROTO=static

IPADDR=172.17.1.1

NETMASK=255.255.255.0

GATEWAY=172.17.1.0

USERCTL=no

TYPE=Bridge

IPV6INIT=no



\[iyunv@Slave1 ~\]\# vim /etc/sysconfig/network-scripts/route-ens32

172.17.2.0/24 via 192.168.115.6 dev ens32



\[iyunv@Slave1 ~\]\# systemctl  restart network.service

 









2）在Slave2宿主机上设置docker容器内网ip网段172.17.2.0/24

\[iyunv@Slave2 ~\]\# vim /proc/sys/net/ipv4/ip\_forward 

1

\[iyunv@Slave2 ~\]\# ovs-vsctl add-br obr0

\[iyunv@Slave2 ~\]\# ovs-vsctl add-port obr0 gre0 -- set Interface gre0 type=gre options:remote\_ip=192.168.115.6



\[iyunv@Slave2 ~\]\# brctl addbr kbr0

\[iyunv@Slave2 ~\]\# brctl addif kbr0 obr0

\[iyunv@Slave2 ~\]\# ip link set dev docker0 down

\[iyunv@Slave2 ~\]\# ip link del dev docker0



\[iyunv@Slave2 ~\] vim /etc/sysconfig/network-scripts/ifcfg-kbr0

ONBOOT=yes

BOOTPROTO=static

IPADDR=172.17.2.1

NETMASK=255.255.255.0

GATEWAY=172.17.2.0

USERCTL=no

TYPE=Bridge

IPV6INIT=no



\[iyunv@Slave2 ~\]\# vim /etc/sysconfig/network-scripts/route-ens32 

172.17.1.0/24 via 192.168.115.5 dev ens32



\[iyunv@Slave2 ~\]\# systemctl  restart network.service



 









三、启动容器测试

Server1和Server2上修改docker启动的虚拟网卡绑定为kbr0，重启docker进程

1）在Server1宿主机上启动容器,然后登陆容器内查看ip，就会发现ip是上面设定额172.17.1.0/24网段的

\[iyunv@Slave1 ~\]\# docker run -idt --name my-server1 daocloud.io/library/centos/bin/bash 



2）在Server2宿主机上启动容器，然后登陆容器内查看ip，就会发现ip是上面设定额172.17.2.0/24网段的

\[iyunv@Slave2 ~\]\#docker run -idt --name my-server1 daocloud.io/library/centos /bin/bash

然后在上面启动的容内互ping对方容器，发现是可以ping通的



