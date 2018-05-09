# Open vSwitch

其实除了上面使用的pipework工具还，还可以使用虚拟交换机\(Open vSwitch\)进行docker容器间的网络通信，废话不多说，下面说下Open vSwitch的使用：

一、在Server1和Server2上分别安装open vswitch

\# yum -y install wget openssl-devel kernel-devel

\# yum groupinstall "Development Tools"

\# adduser ovswitch

\# su - ovswitch

$ wget [http://openvswitch.org/releases/openvswitch-2.3.0.tar.gz](http://openvswitch.org/releases/openvswitch-2.3.0.tar.gz)

$ tar -zxvpf openvswitch-2.3.0.tar.gz

$ mkdir -p ~/rpmbuild/SOURCES

$ sed 's/openvswitch-kmod, //g' openvswitch-2.3.0/rhel/openvswitch.spec openvswitch-2.3.0/rhel/openvswitch\\_no\\_kmod.spec

$ cp openvswitch-2.3.0.tar.gz rpmbuild/SOURCES/

$ rpmbuild -bb --without check ~/openvswitch-2.3.0/rhel/openvswitch\\_no\\_kmod.spec

$ exit

\# yum localinstall /home/ovswitch/rpmbuild/RPMS/x86\\_64/openvswitch-2.3.0-1.x86\\_64.rpm

\# mkdir /etc/openvswitch

\# setenforce 0

\# systemctl start openvswitch.service

\# systemctl status openvswitch.service -l

二、在Slave1和Slave2上建立OVS Bridge并配置路由

1）在Slave1宿主机上设置docker容器内网ip网段172.17.1.0/24

\# vim /proc/sys/net/ipv4/ip\_forward

1

\# ovs-vsctl add-br obr0

\# ovs-vsctl add-port obr0 gre0 -- set Interface gre0 type=gre options:remote\\_ip=192.168.115.5

\# brctl addbr kbr0

\# brctl addif kbr0 obr0

\# ip link set dev docker0 down

\# ip link del dev docker0

\# vim /etc/sysconfig/network-scripts/ifcfg-kbr0

ONBOOT=yes

BOOTPROTO=static

IPADDR=172.17.1.1

NETMASK=255.255.255.0

GATEWAY=172.17.1.0

USERCTL=no

TYPE=Bridge

IPV6INIT=no

\# vim /etc/sysconfig/network-scripts/route-ens32

172.17.2.0/24 via 192.168.115.6 dev ens32

\# systemctl restart network.service

2）在Slave2宿主机上设置docker容器内网ip网段172.17.2.0/24

\# vim /proc/sys/net/ipv4/ip\\_forward

1

\# ovs-vsctl add-br obr0

\# ovs-vsctl add-port obr0 gre0 -- set Interface gre0 type=gre options:remote\\_ip=192.168.115.6

\# brctl addbr kbr0

\# brctl addif kbr0 obr0

\# ip link set dev docker0 down

\# ip link del dev docker0

＃ vim /etc/sysconfig/network-scripts/ifcfg-kbr0

ONBOOT=yes

BOOTPROTO=static

IPADDR=172.17.2.1

NETMASK=255.255.255.0

GATEWAY=172.17.2.0

USERCTL=no

TYPE=Bridge

IPV6INIT=no

\# vim /etc/sysconfig/network-scripts/route-ens32

172.17.1.0/24 via 192.168.115.5 dev ens32

\# systemctl restart network.service

三、启动容器测试

Server1和Server2上修改docker启动的虚拟网卡绑定为kbr0，重启docker进程

1）在Server1宿主机上启动容器,然后登陆容器内查看ip，就会发现ip是上面设定额172.17.1.0/24网段的

\# docker run -idt --name my-server1 daocloud.io/library/centos/bin/bash

2）在Server2宿主机上启动容器，然后登陆容器内查看ip，就会发现ip是上面设定额172.17.2.0/24网段的

\#docker run -idt --name my-server1 daocloud.io/library/centos /bin/bash

然后在上面启动的容内互ping对方容器，发现是可以ping通的

