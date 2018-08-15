pipework是Docker公司工程师Jerome Petazzoni在Github上发布的名为pipework的工具。



号称是容器网络的SDN解决方案，可以在复杂的场景下将容器连接起来。它既支持普通的LXC容器，也支持Docker容器。







其命令行格式如下:



pipework:



Syntax:

pipework &lt;hostinterface&gt; \[-i containerinterface\] \[-l localinterfacename\] \[-a addressfamily\] &lt;guest&gt; &lt;ipaddr&gt;/&lt;subnet&gt;\[@default\_gateway\] \[macaddr\]\[@vlan\]

pipework &lt;hostinterface&gt; \[-i containerinterface\] \[-l localinterfacename\] &lt;guest&gt; dhcp \[macaddr\]\[@vlan\]

pipework route &lt;guest&gt; &lt;route\_command&gt;

pipework rule &lt;guest&gt; &lt;rule\_command&gt;

pipework tc &lt;guest&gt; &lt;tc\_command&gt;

pipework --wait \[-i containerinterface









pipework --wait \[-i containerinterface\]:

这条命令用于等待指定接口真正创建完成。



-i containerinterface: 参数指定要等待的接口名称; 如果没有指定-i参数，则默认等待名为eth1的接口创建完成。







pipework tc &lt;guest&gt; &lt;tc\_command&gt;

用于在指定容器内执行tc流量控制命令



这条命令用于在&lt;guest&gt;指定的容器名所在的网络命名空间中执行流量控制命令&lt;tc\_command&gt;



脚本会查找&lt;guest&gt;容器的pid,并在/var/run/netns下建立相应网络命名空间的符号链接，然后通过ip netns exec在指定网络命名空间中执行tc命令。







pipework rule &lt;guest&gt; &lt;rule\_command&gt;

用于在指定容器内执行ip rule命令。和上面tc的原理类似。







pipework route &lt;guest&gt; &lt;route\_command&gt;

用于在指定容器内执行ip route命令，和上面一条命令原理类似。







pipework \[--direct-phys\] &lt;hostinterface&gt; \[-i containerinterface\] \[-l localinterfacename\] \[-a addressfamily\] &lt;guest&gt; &lt;ipaddr&gt;/&lt;subnet&gt;\[@default\_gateway\] \[macaddr\]\[@vlan\]

pipework \[--direct-phys\] &lt;hostinterface&gt; \[-i containerinterface\] \[-l localinterfacename\] &lt;guest&gt; dhcp \[macaddr\]\[@vlan\]

用于为指定的&lt;guest&gt;容器创建网卡，并桥接到&lt;hostinterface&gt;指定的宿主设备。



容器的网卡名称由\[-i containerinterface\]指定，如果不用-i指定则默认名称为eth1.



&lt;hostinterface&gt;如果已经在宿主机上存在，则有以下几种情况:



是一个linux bridge网桥

是一个openvswitch网桥

是一个ipoib设备

是一个物理网卡

&lt;hostinterface&gt;如果在宿主机上不存在，则会创建。创建的设备有以下几种：



如果名称为"br\*",则创建一个同名的linux bridge

如果名称为"ovs\*",则创建一个同名的openvswitch网桥

dhcp:用于指定容器网卡的ip,也可以指定dhcp来动态获取。



当使用dhcp方式时，需要在主机环境中存在DHCP服务器，同时容器中需要安装dhcp客户端，pipework会根据不同的客户端来执行不同的命令发送dhcp请求。



当指定ip时，ip地址的形式为x.x.x.x/netmask@gateway。\(如:1.1.1.2/24@1.1.1.254\)







macaddr@vlan:用于指定容器网卡的mac地址和所属的vlan.可以单独指定mac或vlan,也可以同时指定。如果需要单独指定vlan,参数形式是@vlanid.



其中mac地址也可以通过md5sum来随机生成，如果要随机生成，此时macaddr的形式是U:macunique,其中macunique为一个唯一的值。



需要注意的是：linux bridge不支持创建vlan,如果要创建vlan只能使用openvswitch或者物理网卡。







物理网卡创建vlan的过程:



&lt;span style="color:\#000000;"&gt;\[ "$IFTYPE" = phys \] && {

  \[ "$VLAN" \] && {

    \[ ! -d "/sys/class/net/${IFNAME}.${VLAN}" \] && {

      ip link add link "$IFNAME" name "$IFNAME.$VLAN" mtu "$MTU" type vlan id "$VLAN"

    }

    ip link set "$IFNAME" up

    IFNAME=$IFNAME.$VLAN

  }

 

  if \[ ! -z "$DIRECT\_PHYS" \]; then

    GUEST\_IFNAME=$IFNAME

  else

    GUEST\_IFNAME=ph$NSPID$CONTAINER\_IFNAME

    ip link add link "$IFNAME" dev "$GUEST\_IFNAME" mtu "$MTU" type macvlan mode bridge

  fi

 

  ip link set "$IFNAME" up

}&lt;/span&gt;



openvswitch创建vlan的过程:



&lt;span style="color:\#000000;"&gt;openvswitch\)

      if ! ovs-vsctl list-ports "$IFNAME" \| grep -q "^${LOCAL\_IFNAME}$"; then

        ovs-vsctl add-port "$IFNAME" "$LOCAL\_IFNAME" ${VLAN:+tag="$VLAN"}

      fi&lt;/span&gt;









\[-a addressfamily\]:参数用于指明ip地址的类型4或者6.



\[-l localinterfacename\]:用于指定创建的veth-pair设备在网桥侧的名称。如果没有指定，则名称为"v+容器中的网卡名+pl+容器的网络空间名".



\[--direct-phys\]:如果我们要创建vlan且使用的是物理网卡，那么默认会在物理网卡上创建一个macvlan设备;如果指定了--direct-phys则直接使用物理网卡不创建macvlan设备。



最后说明的是：为容器添加了网卡和ip后,pipework还会添加对应的路由规则；如果宿主机安装了arping命令还会执行arping向邻居进行一次广播。

