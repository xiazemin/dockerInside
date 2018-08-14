Bridge模式

```
    Bridge模式是Docker默认的网络模式，当Docker进程启动时，会在主机上创建一个名为docker0的虚拟网桥，用来连接宿主机和容器，此主机上的Docker容器都会连接到这个虚拟网桥上，虚拟网桥的工作方式和物理交换机类似，这样所有容器就通过交换机连在了一个二层网络中。
```

Docker利用 veth pair技术，在宿主机上创建了两个虚拟网络接口 veth0 和 veth1（veth pair技术的特性可以保证无论哪一个veth 接收到网络报文，都会无条件地传输给另一方），Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0（容器的网卡，从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关），另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中。可以通过brctl show命令查看，这样容器和网桥就可以相互通信了。网络结构如下图：

![](/assets/importbridge.png)

容器与宿主机通信 : 在桥接模式下，Docker Daemon将veth0附加到docker0网桥上，保证宿主机的报文有能力发往veth0。再将veth1 添加到Docker 容器所属的网络命名空间，保证宿主机的网络报文若发往 veth0 可以立即被 veth1 收到。

容器与外界通信 : 容器如果需要联网，则需要采用 NAT（是一种在 ip 数据包通过路由器或防火墙时，重写来源 ip 地址或目的 ip 地址的技术） 方式。准确的说，是 NATP \(网络地址端口转换\) 方式。NATP 包含两种转换方式：SNAT 和 DNAT 。

宿主机外访问容器时（修改数据包的目的地址）：

![](/assets/importdnat.png)

由于容器的 IP 与端口对外都是不可见的，所以数据包的目的地址为宿主机的 ip 和端口，为 192.168.1.10:24 。

数据包经过路由器发给宿主机 eth0，再经 eth0 转发给 docker0 网桥。由于存在 DNAT 规则，会将数据包的目的地址转换为容器的 ip 和端口，为 172.17.0.n:24 。

宿主机上的 docker0 网桥识别到容器 ip 和端口，于是将数据包发送附加到 docker0 网桥上的 veth0 接口，veth0 接口再将数据包发送给容器内部的 veth1 接口，容器接收数据包并作出响应。

整个过程如下图：

![](/assets/importdnat1.png)

容器访问宿主机之外时（修改数据包的源地址）

![](/assets/importsnat.png)

此时数据包的源地址为容器的 ip 和端口，为 172.17.0.n:24，容器内部的 veth1 接口将数据包发往 veth0 接口，到达 docker0 网桥。

宿主机上的 docker0 网桥发现数据包的目的地址为外界的 IP 和端口，便会将数据包转发给 eth0 ，并从 eth0 发出去。由于存在 SNAT 规则，会将数据包的源地址转换为宿主机的 ip 和端口，为 192.168.1.10:24 。

由于路由器可以识别到宿主机的 ip 地址，所以再将数据包转发给外界，外界接受数据包并作出响应。这时候，在外界看来，这个数据包就是从 192.168.1.10:24 上发出来的，Docker 容器对外是不可见的。

整个过程如下图：

![](/assets/importsnat1.png)

macvlan模式

macvlan本身是linxu kernel的模块，本质上是一种网卡虚拟化技术。其功能是允许在同一个物理网卡上虚拟出多个网卡，通过不同的MAC地址在数据链路层进行网络数据的转发，一块网卡上配置多个 MAC 地址（即多个 interface），每个interface可以配置自己的IP，Docker的macvlan网络实际上就是使用了Linux提供的macvlan驱动

因为多个MAC地址的网络数据包都是从同一块网卡上传输，所以需要打开网卡的混杂模式ip link set eth0 promisc on;

创建macvlan网络不同于桥接模式，需要指定网段和网关，并且都得是真实存在的，例如docker network create -d macvlan --subnet=10.9.8.0/24 --gateway=10.9.8.254 -o parent=eth0 macvlan-test

macvlan模式不依赖网桥，所以brctl show查看并没有创建新的bridge，但是查看容器的网络，会看到虚拟网卡对应了一个interface是2

![](/assets/importmacvlan.png)

查看宿主机的网络，2正是虚机的网卡

![](/assets/importmacvlan2.png)

