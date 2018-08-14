# docker0

docker服务进程在启动的时候会生成一个名为docker0的网桥，容器默认都会挂载到该网桥下，但是我们可以通过添加docker启动参数-b Birdge 或更改docker配置文件来选择使用哪个网桥。

service docker stop //关闭docker服务

ip link set dev docker0 down //关闭docker0网桥

ip link del dev docker0       //删除docker0网桥

自定义网桥设置（/etc/sysconfig/network-scripts/ifcfg-br0文件）

DEVICE="br0"

ONBOOT="yes"

TYPE="Bridge"

BOOTPROTO="static"

IPADDR="10.10.10.20"

NETMASK="255.255.255.0"

GATEWAY="10.10.10.20"

DEFROUTE="yes"

NM\_CONTROLLED="no"

重启网络服务

service network restart

查看网桥

\[black@test opt\]$ brctl show

bridge name     bridge id               STP enabled     interfaces

br0             8000.32e7297502be       no

virbr0          8000.000000000000       yes

接下来我们需要重新启动docker，可以在启动docker服务进程时使用以下两种方式：

第一种：-b 参数指定网桥

docker -d -b br0

docker run -ti --rm centos:latest

第二种：修改/etc/sysconfig/docker文件

我在进行这种操作的时候遇到了一点问题，我修改了/etc/sysconfig/docker文件

\[root@test opt\]\# vi /etc/sysconfig/docker

\# /etc/sysconfig/docker

\# Other arguments to pass to the docker daemon process

\# These will be parsed by the sysv initscript and appended

\# to the arguments list passed to docker -d

接着使用service docker start启动docker服务，但是other\_args并不生效，在centos7下servicer docker start仍然会采用systemctl start docker.service命令来运行，于是我就打开/usr/lib/systemd/system/docker.service查看

\[root@test opt\]\# vi /lib/systemd/system/docker.service

\[Unit\]

Description=Docker Application Container Engine

Documentation=[https://docs.docker.com](https://docs.docker.com)

After=network.target docker.socket

Requires=docker.socket

\[Service\]

ExecStart=/usr/bin/docker -d  -H fd://

MountFlags=slave

LimitNOFILE=1048576

LimitNPROC=1048576

LimitCORE=infinity

\[Install\]

WantedBy=multi-user.target

发现ExecStart一项并没有运行参数，于是将ExecStart改为/usr/bin/docker -d -b br0 -H fd://，运行docker服务，启动一个容器发现能够成功使用br0网桥。

在网上看到了一种更好的方法，将docker.service改为如下

\[black@test ~\]$ vi /usr/lib/systemd/system/docker.service

\[Unit\]

Description=Docker Application Container Engine

Documentation=[https://docs.docker.com](https://docs.docker.com)

After=network.target docker.socket

Requires=docker.socket

\[Service\]

EnvironmentFile=-/etc/sysconfig/docker

ExecStart=/usr/bin/docker -d $other\_args  -H fd://

MountFlags=slave

LimitNOFILE=1048576

LimitNPROC=1048576

LimitCORE=infinity

\[Install\]

WantedBy=multi-user.target

这个时候在other\_args中添加的参数就有效了。

