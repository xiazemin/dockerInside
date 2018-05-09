# pipework实现容器独立ip

pipework 可以在下面用三个场景来使用和工作原理。

环境:安装docker        关闭selinux     开启路由转发（net.ipv4.ip\_forward =  1）

一、将 Docker 容器配置到本地网络环境中

为了使本地网络中的机器和 Docker 容器更方便的通信，我们经常会有将 Docker 容器配置到和主机同一网段的需求。这个需求其实很容易实现，我们只要将 Docker 容器和主机的网卡桥接起来，再给 Docker 容器配上 IP 就可以了。

下面我们来操作一下，我主机 A 地址为 192.168.157.128/24,网关为 192.168.157.2,需要给 Docker容器的地址配置为 192.168.157.150/24。在主机 A 上做如下操作：

安装 pipework

下载地址：wgethttps://github.com/jpetazzo/pipework.git

unzippipework-master.zip

cp -p /root/pipework-master/pipework/usr/local/bin/

启动 Docker 容器。

docker run -itd--name test1 镜像 /bin/bash

配置容器网络，并连到网桥 br0 上。网关在 IP 地址后面加@指定。

pipework br0 test1192.168.157.150/24@192.168.157.2

将主机 eno16777736 桥接到 br0 上，并把 eno16777736 的 IP 配置在 br0 上。

p addradd 192.168.157.128/24 dev br0

ip addrdel 192.168. 157.128/24 dev eno16777736

brctladdif br0 eno16777736

ip routedel default

ip routeadd default via 192.168.157.2 dev br0

注：如果是远程操作，中间网络会断掉，所以放在一条命令中执行。

ip addradd 192.168.157.128/24 dev br0；ip addr del 192.168. 157.128/24 deveno16777736；brctl addif br0 eno16777736；ip route del default；ip route add default via 192.168.157.2dev br0

完成上述步骤后，我们发现 Docker 容器已经可以使用新的 IP 和主机网络里的机器相互通信了

进入容器内部查看容器的地址：

![](https://img-blog.csdn.net/20180118185158325?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

pipework 工作原理分析

那么容器到底发生了哪些变化呢？我们 docker attach 到 test1 上，发现容器中多了一块 eth1的网卡，并且配置了 192.168.157.150/24 的 IP，而且默认路由也改为了 192.168.157.2。这些都是pipework 帮我们配置的。

首先 pipework 检查是否存在 br0 网桥，若不存在，就自己创建。

创建 veth pair 设备，用于为容器提供网卡并连接到 br0 网桥。

使用 docker inspect 找到容器在主机中的 PID，然后通过 PID 将容器的网络命名空间链接到var/run/netns/目录下。这么做的目的是，方便在主机上使用 ip netns 命令配置容器的网络。因为，在 Docker 容器中，我们没有权限配置网络环境。

将之前创建的 veth pair 设备分别加入容器和网桥中。在容器中的名称默认为 eth1，可以通过 pipework 的-i 参数修改该名称。

然后就是配置新网卡的 IP。若在 IP 地址的后面加上网关地址，那么 pipework 会重新配置默认路由。这样容器通往外网的流量会经由新配置的 eth1 出去，而不是通过 eth0 和 docker0。\(若想完全抛弃自带的网络设置，在启动容器的时候可以指定--net=none\)

以上就是 pipework 配置 Docker 网络的过程，这和 Docker 的 bridge 模式有着相似的步骤。事实上，Docker 在实现上也采用了相同的底层机制。

通过源代码，可以看出，pipework 通过封装 Linux 上的 ip、brctl 等命令，简化了在复杂场景下对容器连接的操作命令，为我们配置复杂的网络拓扑提供了一个强有力的工具。当然，如果想了解底层的操作，我们也可以直接使用这些 Linux 命令来完成工作，甚至可以根据自己的需求，添加额外的功能。

二、单主机 Docker 容器 VLAN 划分

pipework 不仅可以使用 Linux bridge 连接 Docker 容器，还可以与 OpenVswitch 结合，实现Docker 容器的 VLAN 划分。下面，就来简单演示一下，在单机环境下，如何实现 Docker 容器间的二层隔离。

为了演示隔离效果，我们将 4 个容器放在了同一个 IP 网段中。但实际他们是二层隔离的两个网络，有不同的广播域。

安装 openvswitch

安装基础环境

yum install gcc makepython-devel openssl-devel kernel-devel graphviz \

kernel-debug-develautoconf automake rpm-build redhat-rpm-config \

libtool

下载 openvswitch 的包

wgethttp://openvswitch.org/releases/openvswitch-2.3.1.tar.gz

![](https://img-blog.csdn.net/20180118185322310?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

解压与打包

tar zxvfopenvswitch-2.3.1.tar.gz

mkdir -p~/rpmbuild/SOURCES

cpopenvswitch-2.3.1.tar.gz ~/rpmbuild/SOURCES/

sed  's/openvswitch-kmod,  //g' openvswitch-2.3.1/rhel/openvswitch.spec &gt;

openvswitch-2.3.1/rhel/openvswitch\_no\_kmod.spec

rpmbuild-bb --without check openvswitch-2.3.1/rhel/openvswitch\_no\_kmod.spec

![](https://img-blog.csdn.net/20180118185304142?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  


之后会在~/rpmbuild/RPMS/x86\_64/里有 2 个文件

![](https://img-blog.csdn.net/20180118185334966?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

安装第一个就行

![](https://img-blog.csdn.net/20180118185345289?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

启动

![](https://img-blog.csdn.net/20180118185409052?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

查看状态

![](https://img-blog.csdn.net/20180118185417822?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到是正常运行状态

安装 pipework

下载地址：wgethttps://github.com/jpetazzo/pipework.git

unzippipework-master.zip

cp -p/root/pipework-master/pipework /usr/local/bin/

创建交换机，把物理网卡加入 ovs1

![](https://img-blog.csdn.net/20180118185446753?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![](https://img-blog.csdn.net/20180118185455364?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![](https://img-blog.csdn.net/20180118185503399?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1eGlhbnNoZW5nMTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在主机 A 上创建 4 个 Docker 容器，test1、test2、test3、test4

docker run -itd--name test1 ubuntu /bin/bash

docker run -itd--name test2 ubuntu /bin/bash

docker run -itd--name test3 ubuntu /bin/bash

docker run -itd--name test4 ubuntu /bin/bash

将 test1，test2 划分到一个 vlan 中，vlan 在 mac 地址后加@指定，此处 mac 地址省略。

pipework ovs1test1 192.168.1.1/24 @100 （注：有空格）

pipework ovs1test2 192.168.1.2/24 @100 （注：有空格）

将 test3，test4 划分到另一个 vlan 中

pipework ovs1test3 192.168.1.3/24 @200 （注：有空格）

pipework ovs1test4 192.168.1.4/24 @200 （注：有空格）

完成上述操作后，使用 docker attach 连到容器中，然后用 ping 命令测试连通性，发现 test1和 test2 可以相互通信，但与 test3 和 test4 隔离。这样，一个简单的 VLAN 隔离容器网络就已经完成。

由于 OpenVswitch 本身支持 VLAN 功能，所以这里 pipework 所做的工作和之前介绍的基本一样，只不过将 Linux bridge 替换成了 OpenVswitch，在将 veth pair 的一端加入 ovs0 网桥时，指定了 tag。底层操作如下：

ovs-vsctladd-port ovs0 veth\* tag=100

