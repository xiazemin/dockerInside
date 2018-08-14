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





