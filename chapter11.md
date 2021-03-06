# 容器间网络通信设置\(Pipework和Open vSwitch\)

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
这个模式下，dokcer不为容器进行任何网络配置。需要我们

自己为容器添加网卡，配置IP。

因此，若想使用pipework配置docker容器的ip地址，必须要在none模式下才可以。

4）其他容器模式（即container模式），--net=container:NAME\\_or\\_ID

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

从上面的网络模型可以看出，容器从原理上是可以与宿主机乃至外界的其他机器通信的。同一宿主机上，容器之间都是连接掉docker0这个网桥上的，它可以作为虚拟交换机使容器可以相互通信。

然而，由于宿主机的IP地址与容器veth pair的 IP地址均不在同一个网段，故仅仅依靠veth par和namespace的技术，还不足以使宿主机以外的网络主动发现容器的存在。为了使外界可以方位容器中的进程，docker采用了端口绑定的方式，也就是通过iptables的NAT，将宿主机上的端口

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

　　
