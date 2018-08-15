DOCKER的内置OVERLAY网络

内置跨主机的网络通信一直是Docker备受期待的功能，在1.9版本之前，社区中就已经有许多第三方的工具或方法尝试解决这个问题，例如Macvlan、Pipework、Flannel、Weave等。

虽然这些方案在实现细节上存在很多差异，但其思路无非分为两种： 二层VLAN网络和Overlay网络

简单来说，二层VLAN网络解决跨主机通信的思路是把原先的网络架构改造为互通的大二层网络，通过特定网络设备直接路由，实现容器点到点的之间通信。这种方案在传输效率上比Overlay网络占优，然而它也存在一些固有的问题。

这种方法需要二层网络设备支持，通用性和灵活性不如后者。

由于通常交换机可用的VLAN数量都在4000个左右，这会对容器集群规模造成限制，远远不能满足公有云或大型私有云的部署需求； 大型数据中心部署VLAN，会导致任何一个VLAN的广播数据会在整个数据中心内泛滥，大量消耗网络带宽，带来维护的困难。

相比之下，Overlay网络是指在不改变现有网络基础设施的前提下，通过某种约定通信协议，把二层报文封装在IP报文之上的新的数据格式。这样不但能够充分利用成熟的IP路由协议进程数据分发；而且在Overlay技术中采用扩展的隔离标识位数，能够突破VLAN的4000数量限制支持高达16M的用户，并在必要时可将广播流量转化为组播流量，避免广播数据泛滥。

因此，Overlay网络实际上是目前最主流的容器跨节点数据传输和路由方案。

容器在两个跨主机进行通信的时候，是使用overlay network这个网络模式进行通信；如果使用host也可以实现跨主机进行通信，直接使用这个物理的ip地址就可以进行通信。overlay它会虚拟出一个网络比如10.0.2.3这个ip地址。在这个overlay网络模式里面，有一个类似于服务网关的地址，然后把这个包转发到物理服务器这个地址，最终通过路由和交换，到达另一个服务器的ip地址。

要实现overlay网络，我们会有一个服务发现。比如说consul，会定义一个ip地址池，比如10.0.2.0/24之类的。上面会有容器，容器的ip地址会从上面去获取。获取完了后，会通过ens33来进行通信，这样就实现跨主机的通信。

环境说明





ss微信截图\_20170918110541.png



\[root@client1 ~\]\# cat /etc/redhat-release 

CentOS Linux release 7.3.1611 \(Core\) 

\[root@client2 ~\]\# cat /etc/redhat-release 

CentOS Linux release 7.3.1611 \(Core\) 

\[root@client1 ~\]\# docker -v 

Docker version 1.12.6, build 78d1802

\[root@client2 ~\]\# docker -v 

Docker version 1.12.6, build 78d1802





修改docker配置



修改它的启动参数，这里的ip等要修改成自己的。

\[root@client1 ~\]\# cat /lib/systemd/system/docker.service \| grep "ExecStart=/usr/bin/dockerd"

ExecStart=/usr/bin/dockerd  -H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock --cluster-store=consul://192.168.6.134:8500 --cluster-advertise=ens33:2376 --insecure-registry=0.0.0.0/0



\[root@client2~\]\# cat /lib/systemd/system/docker.service \| grep "ExecStart=/usr/bin/dockerd"

ExecStart=/usr/bin/dockerd  -H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock --cluster-store=consul://192.168.6.135:8500 --cluster-advertise=ens33:2376 --insecure-registry=0.0.0.0/0





修改完后，需要重启。

\[root@client1 ~\]\# systemctl daemon-reload

\[root@client1 ~\]\# systemctl restart docker



\[root@client2 ~\]\# systemctl daemon-reload

\[root@client2 ~\]\# systemctl restart docker





查看重启后是否启动成功。

\[root@client1 ~\]\# ps aux \| grep dockerd

root     107931  0.4  2.7 507600 27908 ?        Ssl  16:52   0:00 /usr/bin/dockerd -H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock --cluster-store=consul://192.168.6.134:8500 --cluster-advertise=ens33:2376 --insecure-registry=0.0.0.0/0



\[root@client1 ~\]\# ps aux \| grep dockerd

root     107931  0.4  2.7 507600 27908 ?        Ssl  16:52   0:00 /usr/bin/dockerd -H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock --cluster-store=consul://192.168.6.135:8500 --cluster-advertise=ens33:2376 --insecure-registry=0.0.0.0/0





在第一台主机上创建一个consul。

\[root@client1 ~\]\# docker run -d -p 8400:8400 -p 8500:8500 -p 8600:53/udp -h consul progrium/consul -server -bootstrap -ui-dir /ui





查看启动是否成功。

\[root@client1 ~\]\# docker ps 

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                                        NAMES

561ad5f39ca5        progrium/consul     "/bin/start -server -"   27 hours ago        Up 7 seconds        53/tcp, 0.0.0.0:8400-&gt;8400/tcp, 8300-8302/tcp, 8301-8302/udp, 0.0.0.0:8500-&gt;8500/tcp, 0.0.0.0:8600-&gt;53/udp   focused\_spence





创建完后通过浏览器访问一下，可以看到这两台会自动注册上来，这样的话这两个主机之间就会进行通信。



s3.png





在第一台主机上创建一个overlay网络。

\[root@client1 ~\]\# docker network create -d overlay --subnet=10.0.2.1/24 overlay-net 

80e398c37493ec1a4132efa56572a9212ac5688b557772d295c21a0d0916120b

\[root@client1 ~\]\# docker network ls

NETWORK ID          NAME                DRIVER              SCOPE

9008047151be        bridge              bridge              local                             

bd6513b8baec        host                host                local               

6aed7659391a        none                null                local               

80e398c37493        overlay-net         overlay             global  





这边自动回进行通步，因为使用的是同一个服务器发件。

\[root@client2 ~\]\# docker network ls

NETWORK ID          NAME                DRIVER              SCOPE

9008047171be        bridge              bridge              local                             

bd6513b8baec        host                host                local               

6aed7449391a        none                null                local               

80e399c37493        overlay-net         overlay             global





创建一个使用overlay网络的容器。

\[root@client1 ~\]\# docker run -d --name app1 --net=overlay-net registry

c8ec2b34c97abd2e563d236d6fe1b51686f7a9440f6eb171392ca0a6221c2b7a





查看是否创建成功。

root@client1 ~\]\# docker ps 

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                                        NAMES

c8ec2b34c97a        registry            "/entrypoint.sh /etc/"   About an hour ago   Up 6 seconds        5000/tcp                                                                                                     app1





登陆进去查看ip地址是否是10.0.2.0的网段。

\[root@client1 ~\]\# docker exec -it app1 sh

/ \# ifconfig 

eth0      Link encap:Ethernet  HWaddr 02:42:0A:00:02:02  

     inet addr:10.0.2.2  Bcast:0.0.0.0  Mask:255.255.255.0

     inet6 addr: fe80::42:aff:fe00:202%32683/64 Scope:Link

     UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1

     RX packets:15 errors:0 dropped:0 overruns:0 frame:0

     TX packets:8 errors:0 dropped:0 overruns:0 carrier:0

     collisions:0 txqueuelen:0 

     RX bytes:1206 \(1.1 KiB\)  TX bytes:648 \(648.0 B\)



eth1      Link encap:Ethernet  HWaddr 02:42:AC:13:00:02  

     inet addr:172.19.0.2  Bcast:0.0.0.0  Mask:255.255.0.0

     inet6 addr: fe80::42:acff:fe13:2%32683/64 Scope:Link

     UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

     RX packets:14 errors:0 dropped:0 overruns:0 frame:0

     TX packets:14 errors:0 dropped:0 overruns:0 carrier:0

     collisions:0 txqueuelen:0 

     RX bytes:1217 \(1.1 KiB\)  TX bytes:1074 \(1.0 KiB\)



lo        Link encap:Local Loopback  

     inet addr:127.0.0.1  Mask:255.0.0.0

     inet6 addr: ::1%32683/128 Scope:Host

     UP LOOPBACK RUNNING  MTU:65536  Metric:1

     RX packets:4 errors:0 dropped:0 overruns:0 frame:0

     TX packets:4 errors:0 dropped:0 overruns:0 carrier:0

     collisions:0 txqueuelen:1 

     RX bytes:379 \(379.0 B\)  TX bytes:379 \(379.0 B\)





它也具备一个nat网络模式。

/ \# ping www.baidu.com

PING www.baidu.com \(61.135.169.125\): 56 data bytes

64 bytes from 61.135.169.125: seq=0 ttl=127 time=27.659 ms

64 bytes from 61.135.169.125: seq=1 ttl=127 time=29.027 ms

^C

--- www.baidu.com ping statistics ---

2 packets transmitted, 2 packets received, 0% packet loss

round-trip min/avg/max = 27.659/28.343/29.027 ms





在另一台机器上面同样创建一个overlay网路的容器。

\[root@client2 ~\]\# docker run -d --name app2 --net=overlay-net registry

\[root@client2 ~\]\# docker exec -it app2 sh





查看ip地址。

UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1

     RX packets:13 errors:0 dropped:0 overruns:0 frame:0

     TX packets:8 errors:0 dropped:0 overruns:0 carrier:0

     collisions:0 txqueuelen:0 

     RX bytes:1038 \(1.0 KiB\)  TX bytes:648 \(648.0 B\)



eth1      Link encap:Ethernet  HWaddr 02:42:AC:14:00:02  

     inet addr:172.20.0.2  Bcast:0.0.0.0  Mask:255.255.0.0

     inet6 addr: fe80::42:acff:fe14:2%32667/64 Scope:Link

     UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

     RX packets:14 errors:0 dropped:0 overruns:0 frame:0

     TX packets:8 errors:0 dropped:0 overruns:0 carrier:0

     collisions:0 txqueuelen:0 

     RX bytes:1080 \(1.0 KiB\)  TX bytes:648 \(648.0 B\)



lo        Link encap:Local Loopback  

     inet addr:127.0.0.1  Mask:255.0.0.0

     inet6 addr: ::1%32667/128 Scope:Host

     UP LOOPBACK RUNNING  MTU:65536  Metric:1

     RX packets:0 errors:0 dropped:0 overruns:0 frame:0

     TX packets:0 errors:0 dropped:0 overruns:0 carrier:0

     collisions:0 txqueuelen:1 

     RX bytes:0 \(0.0 B\)  TX bytes:0 \(0.0 B\)





查看client1和client2的容器网络是否可以互通。

/ \# ping 10.0.2.2

PING 10.0.2.2 \(10.0.2.2\): 56 data bytes

64 bytes from 10.0.2.2: seq=0 ttl=64 time=0.748 ms

64 bytes from 10.0.2.2: seq=1 ttl=64 time=0.428 ms

64 bytes from 10.0.2.2: seq=2 ttl=64 time=1.073 ms

^C

--- 10.0.2.2 ping statistics ---

3 packets transmitted, 3 packets received, 0% packet loss

round-trip min/avg/max = 0.428/0.749/1.073 ms



/ \# ping app1

PING app1 \(10.0.2.2\): 56 data bytes

64 bytes from 10.0.2.2: seq=0 ttl=64 time=0.418 ms

64 bytes from 10.0.2.2: seq=1 ttl=64 time=0.486 ms

64 bytes from 10.0.2.2: seq=2 ttl=64 time=0.364 ms

^C

--- app1 ping statistics ---

3 packets transmitted, 3 packets received, 0% packet loss

round-trip min/avg/max = 0.364/0.422/0.486 ms





到此， 在client2主机上的容器可以直接互通。我们这里实现了跨主机通信，是通过overlay network这种网络模式进行通信的。

