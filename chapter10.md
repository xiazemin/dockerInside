# 利用pipework为docker容器设置固定IP

bridge网络模式为每一个启动的容器动态分配一个IP，以172.17.0.1为网关，172.17.0.2,172.17.0.3依次类推作为容器的ip,这样也算是



每个容器有了ip,当下次启动多个容器的时候IP还是会按照这种方式分配，表面上还是一个固定IP的方式，但是这种方式对容器启动顺序



有严格的要求。还有一种方式就是通过人为指定IP的方式，这种方式就是今天讲的利用pipework为容器指定IP。



启动容器时可以通过--net=none指定容器网络模式。有四种网络模式可以选择:host,container,none,bridge\(默认\)，如果不设置这个参数，



那么默认就是bridge的模式，采用dhcp的方式分配IP。使用pipework为容器指定ip,对宿主机有要求，需要宿主机有网桥，对redhat7可以



使用yum install bridge-utils来安装网桥，安装完成之后设置网桥的IP就可以了。



下面一步一步来通过pipework为docker容器指定ip\(前提是机器上已经安装了docker服务，并且服务开启，有镜像可使用\):



第一步、安装网桥设备;



yum install -y bridge-utils



安装完毕即可在命令行下通过brctl 命令查看网桥



第二步、设置网络地址,我们假定宿主机是固定IP，并设置了网关;



?

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

vi /etc/sysconfig/network-scripts/ifcfg-eno16777728

 

TYPE=Ethernet

 

BOOTPROTO=none

 

DEFROUTE=yes

 

PEERROUTES=yes

 

PEERDNS=yes

 

NAME=eno16777728

 

UUID=00f2e830-ed07-4ace-9e54-56a325e3a690

 

ONBOOT=yes

 

\#IPADDR0=192.168.61.150

 

\#PREFIX0=24

 

\#GATEWAY0=192.168.61.2

 

\#DNS1=192.168.61.2

 

HWADDR=00:0C:29:F7:22:81

 

BRIDGE="br-ex"

 

vi /etc/sysconfig/network-scripts/ifcfg-br-ex

 

TYPE=Bridge

 

BOOTPROTO=static

 

IPADDR=192.168.61.150

 

NETMASK=255.255.255.0

 

GATEWAY=192.168.61.2

 

PREFIX=24

 

DNS1=192.168.61.2

 

NAME=br-ex

 

ONBOOT=yes

 

DEVICE=br-ex

以上修改即为设置IP，设置完毕可以通过命令service network restart重启网络



?

1

2

3

4

5

6

7

8

9

\[root@docker ~\]\# brctl show

 

bridge name bridge id STP enabled interfaces

 

br-ex 8000.000c29f72281 no eno16777728

 

veth1pl90365

 

docker0 8000.0242dda719e2 no

第三步、启动docker容器并指定网络模式为none



?

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

\[root@docker ~\]\# docker images

 

REPOSITORY TAG IMAGE ID CREATED SIZE

 

docker.io/zookeeper latest 19604ac4a163 Less than a second ago 143 MB

 

redis latest 07818b5b6de8 8 hours ago 482.9 MB

 

\[root@docker ~\]\# docker run -it -d --net=none --name ip-test redis /bin/bash

 

\[root@docker ~\]\# docker ps

 

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

 

4190f301a367 redis "/bin/bash" 55 minutes ago Up 55 minutes ip-test

第四步、获取pipework可执行程序



?

1

2

3

4

5

6

7

8

9

10

11

\[root@docker ~\]\# git clone https://github.com/jpetazzo/pipework.git

 

\[root@docker ~\]\# cd pipework/

 

\[root@docker pipework\]\# ls

 

docker-compose.yml doctoc LICENSE pipework pipework.spec README.md

 

\[root@docker ~\]\# cd ..

 

\[root@docker ~\]\# cp -rp pipework/pipework /usr/local/bin

第五步、设置docker容器IP



?

1

\[root@docker ~\]\# pipework br-ex ip-test 192.168.61.100/24@192.168.61.2

第六步、验证IP设置是否正确



?

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

\[root@docker ~\]\# ping 192.168.61.100

 

PING 192.168.61.100 \(192.168.61.100\) 56\(84\) bytes of data.

 

64 bytes from 192.168.61.100: icmp\_seq=1 ttl=64 time=0.206 ms

 

64 bytes from 192.168.61.100: icmp\_seq=2 ttl=64 time=0.081 ms

 

^C

 

--- 192.168.61.100 ping statistics ---

 

2 packets transmitted, 2 received, 0% packet loss, time 1001ms

 

rtt min/avg/max/mdev = 0.081/0.143/0.206/0.063 ms

补充：



1、设置宿主机网桥IP地址的时候就是将原来网卡eno16777728的固定IP地址设置到br-ex网桥上，把网卡eno16777728IP和网关子网掩码



等都注释掉。有的机器上网卡可能叫eno16777736,根据具体网卡来修改配置文件。



2、网桥br-ex的名字可以随便叫,叫br0也可以，就是对应配置文件名称就必须是ifcfg-br0，类型为Bridge。



3、网桥设置的网关，我的机器是192.168.61.2,这个是安装虚拟机时vm默认设置的，有的机器网关为192.168.xx.1，最后对docker容器设置



IP时需要指定192.168.xx.yy/24@192.168.xx.1,@后面就是指定宿主机的网关，这里一定要注意，否则网络不通。

