xhyve

xhyve是一个在原生OS X Hypervisor框架上封装的极其酷的后台虚拟技术。我们不再需要再安装笨重的VirtualBox、VMWare Fusion或者Parallels Desktop来作为“边车”在Mac上运行Linux程序。

在Dockercon2017上发布了一个能够在容器内创建最小Linux OS 镜像的架构——LinuxKit

安装Xhyve



第一步，需要安装Xhyve。Xhyve是一个在OSx’s Hypervisor运行的虚拟机操作系统，其架构允许在用户空间内运行虚拟机。这是Docker为MAC用户准备的。有很多方法可以做到这一点，但最简单的莫过于用Homebrew。在机器上安装——



$ brew update



$ brew install --HEAD xhyve



安装完成后，检验是否按照如下代码运行：



$ xhyve -h



如果看到包含各种可用 flags 的输出，那么可以准备进入下一步。



2



构建Moby



第二步是构建Moby工具。此工具会提供读取。稍后会详细说明Yaml的功能点，执行多样化的Docker命令来建立Linux OS,如果喜欢，现在就可以运行Image。



这比Moby’s Hyperkit项目要方便。建立Kernel和Initrdimages会比 a qcow image 耗时更少，所以，如果需要快速的重复Xhyve,这是一个不错的实现方式。



首先，CD 到任意一个喜欢的工作目录，并Clone LinuxKit repo：



$ git clone https://github.com/linuxkit/linuxkit.git



接下来安装Moby binary：



$ cd linuxkit



$ make



$ sudo make install



完成后，需要准备建立使用LinuxKit的首个Linux image。



3



定制和构建Linux映像



完成这些先决条件后，先建立首个镜像。LinuxKit项目包括一些例子，其中某些例子也在Dockercon做了现场Demo。在Onboot建立一个能够启动Redis instance的简单镜像，这样比从复杂的角度入手更快！



首先，启动一个能够描述Linux image的 Yaml file，接着拉取实例，将会拿到基础的Docker image，并且给Redis 增加一个入口。



$ vi linux-redis.yaml



kernel:



image: "linuxkit/kernel:4.9.x"



cmdline: "console=ttyS0 console=tty0 page\_poison=1"



init:



- linuxkit/init:63eed9ca7a09d2ce4c0c5e7238ac005fa44f564b



- linuxkit/runc:b0fb122e10dbb7e4e45115177a61a3f8d68c19a9



- linuxkit/containerd:18eaf72f3f4f9a9f29ca1951f66df701f873060b



- linuxkit/ca-certificates:eabc5a6e59f05aa91529d80e9a595b85b046f935



onboot:



- name: sysctl



image: "linuxkit/sysctl:1f5ec5d5e6f7a7a1b3d2ff9dd9e36fd6fb14756a"



net: host



pid: host



ipc: host



capabilities:



- CAP\_SYS\_ADMIN



readonly: true



- name: sysfs



image: linuxkit/sysfs:6c1d06f28ddd9681799d3950cddf044b930b221c



- name: binfmt



image: "linuxkit/binfmt:8881283ac627be1542811bd25c85e7782aebc692"



binds:



- /proc/sys/fs/binfmt\_misc:/binfmt\_misc



readonly: true



- name: format



image: "linuxkit/format:53748000acf515549d398e6ae68545c26c0f3a2e"



binds:



- /dev:/dev



capabilities:



- CAP\_SYS\_ADMIN



- CAP\_MKNOD



- name: mount



image: "linuxkit/mount:d2669e7c8ddda99fa0618a414d44261eba6e299a"



binds:



- /dev:/dev



- /var:/var:rshared,rbind



capabilities:



- CAP\_SYS\_ADMIN



rootfsPropagation: shared



command: \["/mount.sh", "/var/lib/docker"\]



services:



- name: rngd



image: "linuxkit/rngd:3dad6dd43270fa632ac031e99d1947f20b22eec9"



capabilities:



- CAP\_SYS\_ADMIN



oomScoreAdj: -800



readonly: true



- name: dhcpcd



image: "linuxkit/dhcpcd:57a8ef29d3a910645b2b24c124f9ce9ef53ce703"



binds:



- /var:/var



- /tmp/etc:/etc



capabilities:



- CAP\_NET\_ADMIN



- CAP\_NET\_BIND\_SERVICE



- CAP\_NET\_RAW



net: host



oomScoreAdj: -800



- name: ntpd



image: "linuxkit/openntpd:a570316d7fc49ca1daa29bd945499f4963d227af"



capabilities:



- CAP\_SYS\_TIME



- CAP\_SYS\_NICE



- CAP\_SYS\_CHROOT



- CAP\_SETUID



- CAP\_SETGID



net: host



- name: redis



image: "redis:3.0.7-alpine"



capabilities:



- CAP\_NET\_BIND\_SERVICE



- CAP\_CHOWN



- CAP\_SETUID



- CAP\_SETGID



- CAP\_DAC\_OVERRIDE



net: host



files:



- path: etc/docker/daemon.json



contents: '{"debug": true}'



trust:



image:



- linuxkit/kernel



outputs:



- format: kernel+initrd



现在快速地查看下这个文件。在没有展开更详尽的介绍之前（那些在LinuxKit git repo 中可以看到的），我们将聚焦在以下要点上——



开始的部分是“基础”镜像，定义了Kernel和Kernel命令行参数；



紧接着，是一些Init所需的 OCI兼容的LinuxKit镜像；在此之后，是Onboot部分，里面描述的是另外一些需要再其他服务启动之前由RunC顺序启动的基础镜像；



再之后就是服务（Services）了，同样是OCI兼容的镜像，由Containerd启动并保持运行状态。



最后，指定构建过程中Moby将要创建的输出文件信息。下面，构建镜像。



$ moby build linux-redis



下列是可以看到的输出信息——



Extract kernel image: linuxkit/kernel:4.9.x



Add init containers:



Process init image: linuxkit/init:63eed9ca7a09d2ce4c0c5e7238ac005fa44f564b



Process init image: linuxkit/runc:b0fb122e10dbb7e4e45115177a61a3f8d68c19a9



Process init image: linuxkit/containerd:18eaf72f3f4f9a9f29ca1951f66df701f873060b



Process init image: linuxkit/ca-certificates:eabc5a6e59f05aa91529d80e9a595b85b046f935



Add onboot containers:



Create OCI config for linuxkit/sysctl:1f5ec5d5e6f7a7a1b3d2ff9dd9e36fd6fb14756a



Create OCI config for linuxkit/sysfs:6c1d06f28ddd9681799d3950cddf044b930b221c



Create OCI config for linuxkit/binfmt:8881283ac627be1542811bd25c85e7782aebc692



Create OCI config for linuxkit/format:53748000acf515549d398e6ae68545c26c0f3a2e



Create OCI config for linuxkit/mount:d2669e7c8ddda99fa0618a414d44261eba6e299a



Add service containers:



Create OCI config for linuxkit/rngd:3dad6dd43270fa632ac031e99d1947f20b22eec9



Create OCI config for linuxkit/dhcpcd:57a8ef29d3a910645b2b24c124f9ce9ef53ce703



Create OCI config for linuxkit/openntpd:a570316d7fc49ca1daa29bd945499f4963d227af



Create OCI config for redis:3.0.7-alpine



Add files:



etc/docker/daemon.json



Create outputs:



linux-redis-bzImage linux-redis-initrd.img linux-redis-cmdline



以下是创建的文件：



The raw kernel image \(linux-redis-bzImage\)



The init ramdisk \(linux-redis-initrd.img\)



The commandline options youll need to provide to xhyve in a file



4



运行和输出



现在已经建立镜像，准备运行。



首先，必须要创建一个告诉Xhyve如何实例镜像作为虚拟机的脚本。在这之前，着重强调，可以通过Moby工具输出一个Qcow image。



接着使用 Moby run如同VM一样运行Image。推荐使用Xhyve，因为现在没有其他的Non-linuxkt操作系统能够在Hypervisor架构上运行。



下面需要做一件事来运行镜像：一个为了建立虚拟机而定义的Xhyve参数的本。在这个脚本中主要的项目是从Moby构建流程中获取的。



下面是一个启动Redis server的例子：



$ vi linux-redis.sh



\#!/bin/sh



KERNEL="linux-redis-bzImage"



INITRD="linux-redis-initrd.img"



CMDLINE="console=ttyS0 console=tty0 page\_poison=1"



MEM="-m 1G"



PCI\_DEV="-s 0:0,hostbridge -s 31,lpc"



LPC\_DEV="-l com1,stdio"



ACPI="-A"



\#SMP="-c 2"



\# sudo if you want networking enabled



NET="-s 2:0,virtio-net"



xhyve $ACPI $MEM $SMP $PCI\_DEV $LPC\_DEV $NET -f kexec,$KERNEL,$INITRD,"$CMDLINE"



一旦文件已经创建，使之可执行并且运行它。目前的环节中有些需要注意的事项：



1、如果有任何VPN连接到无线网络上，请关掉它。因为当VPN运行时会有一些已知的流量路径问题。



2、如果准备网络连接虚拟机，需要执行一些超级用户的脚本。



当没问题后运行它：



$ chown 755 linux-redis.sh



$ sudo ./linux-redis.sh



当虚拟机运行的时候将有一组Output会Fly，最后，在新虚拟机上得到一个命令行：



Welcome to LinuxKit



\#\# .



\#\# \#\# \#\# ==



\#\# \#\# \#\# \#\# \#\# ===



/"""""""""""""""""\_\_\_/ ===



~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ / ===- ~~~



\_\_\_\_\_\_ o \_\_/



\_\_/



\_\_\_\_\_\_\_\_\_\_\_/



/ \# \[ 2.125127\] IPVS: Creating netns size=2104 id=1



\[ 2.125466\] IPVS: ftp: loaded support on port\[0\] = 21



\[ 2.156114\] IPVS: Creating netns size=2104 id=2



\[ 2.156496\] IPVS: ftp: loaded support on port\[0\] = 21



\[ 2.177714\] tsc: Refined TSC clocksource calibration: 2193.340 MHz



\[ 2.178170\] clocksource: tsc: mask: 0xffffffffffffffff max\_cycles: 0x1f9d9f9c94d, max\_idle\_ns: 440795310624 ns



\[ 2.399509\] IPVS: Creating netns size=2104 id=3



\[ 2.400027\] IPVS: ftp: loaded support on port\[0\] = 21



\[ 2.670029\] IPVS: Creating netns size=2104 id=4



\[ 2.670555\] IPVS: ftp: loaded support on port\[0\] = 21



\[ 2.773492\] random: dhcpcd: uninitialized urandom read \(112 bytes read\)



\[ 2.791653\] random: redis-server: uninitialized urandom read \(19 bytes read\)



\[ 2.792066\] random: redis-server: uninitialized urandom read \(1024 bytes read\)



\[ 2.911251\] IPVS: Creating netns size=2104 id=5



\[ 2.911770\] IPVS: ftp: loaded support on port\[0\] = 21



\[ 2.935150\] random: rngd: uninitialized urandom read \(16 bytes read\)



\[ 2.955187\] random: crng init done



\[ 3.187797\] clocksource: Switched to clocksource tsc



/ \#



当Redis server正在运行时，看看发生了什么：



/ \# netstat -an



Active Internet connections \(servers and established\)



Proto Recv-Q Send-Q Local Address Foreign Address State



tcp 0 0 0.0.0.0:6379 0.0.0.0:\* LISTEN



tcp 0 0 :::6379 :::\* LISTEN



udp 0 0 192.168.64.17:44773 52.6.160.3:123 ESTABLISHED



udp 0 0 192.168.64.17:44091 208.75.89.4:123 ESTABLISHED



udp 0 0 0.0.0.0:68 0.0.0.0:\*



udp 0 0 192.168.64.17:33429 192.96.202.120:123 ESTABLISHED



udp 0 0 192.168.64.17:39584 69.89.207.99:123 ESTABLISHED



raw 0 0 ::%192:58 ::%32631:\* 58



Active UNIX domain sockets \(servers and established\)



Proto RefCnt Flags Type State I-Node Path



unix 2 \[ ACC \] STREAM LISTENING 14907 /var/run/dhcpcd.sock



unix 2 \[ ACC \] STREAM LISTENING 14909 /var/run/dhcpcd.unpriv.sock



unix 2 \[ ACC \] STREAM LISTENING 14248 /run/containerd/debug.sock



unix 2 \[ ACC \] STREAM LISTENING 14258 /run/containerd/containerd.sock



unix 2 \[ ACC \] STREAM LISTENING 15051 /var/run/ntpd.sock



unix 3 \[ \] STREAM CONNECTED 15055



unix 3 \[ \] STREAM CONNECTED 15050



unix 2 \[ \] DGRAM 15025



unix 3 \[ \] STREAM CONNECTED 15054



unix 3 \[ \] STREAM CONNECTED 15049



/ \#



看起来机器的Redis默认端口是6379。现在，看它是否在网络上可以被看到和获取。首先，找到虚拟机的IP地址：



/ \# ip a



1: lo: &lt;LOOPBACK,UP,LOWER\_UP&gt; mtu 65536 qdisc noqueue state UNKNOWN qlen 1



link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00



inet 127.0.0.1/8 brd 127.255.255.255 scope host lo



valid\_lft forever preferred\_lft forever



inet6 ::1/128 scope host



valid\_lft forever preferred\_lft forever



2: bond0: &lt;BROADCAST,MULTICAST400&gt; mtu 1500 qdisc noop state DOWN qlen 1000



link/ether ca:41:0c:a4:ea:c2 brd ff:ff:ff:ff:ff:ff



3: dummy0: &lt;BROADCAST,NOARP&gt; mtu 1500 qdisc noop state DOWN qlen 1000



link/ether 1a:23:2d:47:af:d5 brd ff:ff:ff:ff:ff:ff



4: eth0: &lt;BROADCAST,MULTICAST,UP,LOWER\_UP&gt; mtu 1500 qdisc pfifo\_fast state UP qlen 1000



link/ether f2:94:56:b6:96:93 brd ff:ff:ff:ff:ff:ff



inet 192.168.64.17/24 brd 192.168.64.255 scope global eth0



valid\_lft forever preferred\_lft forever



inet6 fe80::f094:56ff:feb6:9693/64 scope link



valid\_lft forever preferred\_lft forever



5: teql0: mtu 1500 qdisc noop state DOWN qlen 100



link/void



6: tunl0@NONE: mtu 1480 qdisc noop state DOWN qlen 1



link/ipip 0.0.0.0 brd 0.0.0.0



7: gre0@NONE: mtu 1476 qdisc noop state DOWN qlen 1



link/gre 0.0.0.0 brd 0.0.0.0



8: gretap0@NONE: &lt;BROADCAST,MULTICAST&gt; mtu 1462 qdisc noop state DOWN qlen 1000



link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff



9: ip\_vti0@NONE: mtu 1332 qdisc noop state DOWN qlen 1



link/ipip 0.0.0.0 brd 0.0.0.0



10: ip6\_vti0@NONE: mtu 1500 qdisc noop state DOWN qlen 1



link/tunnel6 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00 brd 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00



11: sit0@NONE: mtu 1480 qdisc noop state DOWN qlen 1



link/sit 0.0.0.0 brd 0.0.0.0



12: ip6tnl0@NONE: mtu 1452 qdisc noop state DOWN qlen 1



link/tunnel6 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00 brd 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00



13: ip6gre0@NONE: mtu 1448 qdisc noop state DOWN qlen 1



link/\[823\] 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00 brd 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00



ip地址是192.168.64.17，从Max OS X主机，使用Netcat测试连接状态和服务器状态：



$ nc 192.168.64.17 6379



ping



+PONG



一旦完成Poking around，键入 halt 关闭虚拟机



/ \# halt



现在我们就有了一个运行在 OS X 虚拟机中的可用的 LinuxKit image。



然而这也仅仅只是一个开始，可以用这个模板来创建其他镜像，定制你想要的东西。同时，不是一定需要使用Redis，尝试替换成你自己服务的 Docker images，启动虚拟机。

