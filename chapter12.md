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

\# docker inspect --format='{{.NetworkSettings.IPAddress}}' 224facf8e054

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





