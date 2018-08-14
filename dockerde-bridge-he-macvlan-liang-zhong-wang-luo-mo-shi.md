Bridge模式    



        Bridge模式是Docker默认的网络模式，当Docker进程启动时，会在主机上创建一个名为docker0的虚拟网桥，用来连接宿主机和容器，此主机上的Docker容器都会连接到这个虚拟网桥上，虚拟网桥的工作方式和物理交换机类似，这样所有容器就通过交换机连在了一个二层网络中。



Docker利用 veth pair技术，在宿主机上创建了两个虚拟网络接口 veth0 和 veth1（veth pair技术的特性可以保证无论哪一个veth 接收到网络报文，都会无条件地传输给另一方），Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0（容器的网卡，从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关），另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中。可以通过brctl show命令查看，这样容器和网桥就可以相互通信了。网络结构如下图：

![](/assets/importbridge.png)

