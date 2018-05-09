# docker跨主机容器通信

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

\# brctl show

查看当前 docker0 ip

\# ifconfig

在容器运行时，每个容器都会分配一个特定的虚拟机口并桥接到docker0。每个容器都会配置同docker0 ip相同网段的专用ip 地址，docker0的IP地址被用于所有容器的默认网关。

一般启动的容器中ip默认是172.17.0.1/24网段的。

\# docker run -t -i --name my-test centos /bin/bash

\# docker inspect c5217f7bd44c \|grep IPAddress

那么能不能在创建容器的时候指定特定的ip呢？这是当然可以实现的！

注意：宿主机的ip路由转发功能一定要打开，否则所创建的容器无法联网！

\# cat /proc/sys/net/ipv4/ip\_forward  1

一、创建容器使用特定范围的IP

Docker 会尝试寻找没有被主机使用的ip段，尽管它适用于大多数情况下，但是它不是万能的，有时候我们还是需要对ip进一步规划。

Docker允许你管理docker0桥接或者通过-b选项自定义桥接网卡，需要安装bridge-utils软件包。操作流程如下：

a）确保docker的进程是停止的

b）创建自定义网桥

c）给网桥分配特定的ip

d）以-b的方式指定网桥

具体操作过程如下（比如创建容器的时候，指定ip为192.168.5.1/24网段的）：

\# ip link set dev docker0 down

\# brctl delbr docker0

\# brctl addbr bridge0

\# ip addr add 192.168.5.1/24 dev bridge0 //注意，这个192.168.5.1就是所建容器的网关地址。通过docker inspect container\_id能查看到

\# ip link set dev bridge0 up

\# ip addr show bridge0

\# vim /etc/sysconfig/docker //即将虚拟的桥接口由默认的docker0改为bridge0

将

OPTIONS='--selinux-enabled --log-driver=journald'

改为

OPTIONS='--selinux-enabled --log-driver=journald -b=bridge0' //即添加-b=bridge0

\# service docker restart

\# docker inspect --format='\[\[.NetworkSettings.IPAddress}}' 224facf8e054

192.168.5.3

然后创建容器，查看下容器ip是否为设定的192.168.5.1/24网段的

\# brctl show

bridge name bridge id STP enabled interfaces

bridge0 8000.ba141fa20c91 no vethe7e227b

vethf382771

使用pipework给容器设置一个固定的ip

可以利用pipework为容器指定一个固定的ip，操作方法非常简单，如下：

\# brctl addbr br0

\# ip link set dev br0 up

\# ip addr add 192.168.114.1/24 dev br0 //这个ip相当于br0网桥的网关ip，可以随意设定。

\# docker run -ti -d --net=none --name=my-test1 docker.io/nginx /bin/bash

\# pipework br0 -i eth0 my-test1 192.168.114.100/24@192.168.114.1

\# docker exec -ti my-test1 /bin/bash

\# ip addr

my-test1容器和my-test2容器在同一个宿主机上，所以它们固定后的ip是可以相互ping通的，如果是在不同的宿主机上，则就无法ping通！

所以说：

这样使用pipework指定固定ip的容器，在同一个宿主机下的容器间的ip是可以相互ping通的，但是跨主机的容器通过这种方式固定ip后就不能ping通了。

跨主机的容器间的通信可以看下面的介绍。

二、不同主机间的容器通信（pipework config docker container ip）

我的centos7测试机上的docker是yum安装的，默认自带pipework工具，所以就不用在另行安装它了。

如果没有pipework工具，可以安装下面步骤进行安装：

\# git clone [https://github.com/jpetazzo/pipework.git](https://github.com/jpetazzo/pipework.git)

\# sudo cp -rp pipework/pipework /usr/local/bin/

安装相应依赖软件\(网桥

\#sudo apt-get install iputils-arping bridge-utils -y

查看Docker宿主机上的桥接网络

\# brctl show

bridge name bridge id STP enabled interfaces

docker0 8000.02426f15541e no veth92d132f

有两种方式做法：

1）可以选择删除docker0，直接把docker的桥接指定为br0

2）也可以选择保留使用默认docker0的配置，这样单主机容器之间的通信可以通过docker0

跨主机不同容器之间通过pipework将容器的网卡桥接到br0上，这样跨主机容器之间就可以通信了。

如果保留了docker0，则容器启动时不加--net=none参数，那么本机容器启动后就是默认的docker0自动分配的ip（默认是172.17.1.0/24网段），它们之间是可以通信的

跨宿主机的容器创建时要加--net=none参数，待容器启动后通过pipework给容器指定ip，这样跨宿主机的容器ip是在同一网段内的同网段地址，因此可以通信。

一般来说：最好在创建容器的时候加上--net=none，防止自动分配的IP在局域网中有冲突。若是容器创建后自动获取ip，下次容器启动会ip有变化，可能会和物理网段中的ip冲突

实例说明如下：







宿主机信息







ip：192.168.1.23 （网卡设备为eth0）







gateway：192.168.1.1







netmask：255.255.255.0







1）删除虚拟桥接卡docker0的配置





\# service docker stop



\# ip link set dev docker0 down



\# brctl delbr docker0







\# brctl addbr br0





\# ip link set dev br0 up







\# ip addr del 192.168.1.23/24 dev eth0//删除宿主机网卡的IP（如果是使用这个地址进行的远程连接，这一步操作后就会断掉；如果是使用外网地址连接的话，就不会断开）







\# ip addr add 192.168.1.23/24 dev br0 //将宿主主机的ip设置到br0





\# brctl addif br0 eth0 //将宿主机网卡挂到br0上



\# ip route del default //删除默认的原路由，其实就是eth0上使用的原路由192.1681.1（这步小心，注意删除后要保证机器能远程连接上，最好是通过外网ip远程连的。别删除路由后，远程连接不上，中断了）





\# ip route add default via 192.168.1.1 dev br0 //为br0设置路由





\# vim /etc/sysconfig/docker //即将虚拟的桥接口由默认的docker0改为bridge0







将







OPTIONS='--selinux-enabled --log-driver=journald'







改为







OPTIONS='--selinux-enabled --log-driver=journald -b=br0' //即添加-b=br0





\# service docker start







启动一个手动设置网络的容器







\# docker ps







CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES







6e64eade06d1 docker.io/centos "/bin/bash" 10 seconds ago Up 9 seconds my-centos



\# docker run -itd --net=none --name=my-test1 docker.io/centos







为my-test1容器设置一个与桥接物理网络同地址段的ip（如下，"ip@gateway"







默认不指定网卡设备名，则默认添加为eth0。可以通过-i参数添加网卡设备名



\# pipework br0 -i eth0 my-test1 192.168.1.190/24@192.168.1.1







同理，在其他机器上启动容器，并类似上面用pipework设置一个同网段类的ip，这样跨主机的容器就可以相互ping通了！





2）保留默认虚拟桥接卡docker0的配置



\# cd /etc/sysconfig/network-scripts/



\# cp ifcfg-eth0 ifcfg-eth0.bak

\# cp ifcfg-eth0 ifcfg-br0



\# vim ifcfg-eth0 //增加BRIDGE=br0，删除IPADDR,NETMASK,GATEWAY,DNS的设置



......



BRIDGE=br0



\# vim ifcfg-br0 //修改DEVICE为br0,Type为Bridge,把eth0的网络设置设置到这里来（里面应该有ip，网关，子网掩码或DNS设置）



......



TYPE=Bridge



DEVICE=br0



\# service network restart



\# service docker restart



开启一个容器并指定网络模式为none（这样，创建的容器就不会通过docker0自动分配ip了，而是根据pipework工具自定ip指定）



\# docker run -itd --net=none --name=my-centos docker.io/centos /bin/bash



6e64eade06d1eb20be3bd22ece2f79174cd033b59182933f7bbbb502bef9cb0f



接着给容器配置网络



\# pipework br0 -i eth0 my-centos 192.168.1.150/24@192.168.1.1



\# docker attach 6e64eade06d1



\# ifconfig eth0 //若没有ifconfig命令，可以yum安装net-tools工具



\# route -n



Kernel IP routing table



Destination Gateway Genmask Flags Metric Ref Use Iface



0.0.0.0 192.168.1.1 0.0.0.0 UG 0 0 0 eth0



192.168.115.0 0.0.0.0 255.255.255.0 U 0 0 0 eth0



另外pipework不能添加静态路由，如果有需求则可以在run的时候加上--privileged=true 权限在容器中手动添加，但这种方法安全性有缺陷。



除此之外，可以通过ip netns（--help参考帮助）添加静态路由，以避免创建容器使用--privileged=true选项造成一些不必要的安全问题：



如下获取指定容器的pid



\# docker inspect --format="［［ .State.Pid }}" 6e64eade06d1



7852

\# ln -s /proc/7852/ns/net /var/run/netns/7852



\# ip netns exec 7852 ip route add 192.168.0.0/16 dev eth0 via 192.168.1.1



\# ip netns exec 7852 ip route //添加成功



192.168.0.0/16 via 192.168.1.1 dev eth0



同理，在其它宿主机进行相应的配置，新建容器并使用pipework添加虚拟网卡桥接到br0，如此创建的容器间就可以相互通信了。



