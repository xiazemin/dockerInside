\# 在network节点上添加一个名为nstest的namespace

\[root@network ~\]\# ip netns add nstest

\[root@network ~\]\#

\# 查看namespace中的网络信息

\[root@network ~\]\# ip netns exec nstest ip addr

1: lo: &lt;LOOPBACK&gt; mtu 65536 qdisc noop state DOWN

    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

\[root@network ~\]\#

\# 创建一对虚拟网卡veth-a 和 veth-b，两者由一根虚拟网线连接

\[root@network ~\]\# ip link add veth-a type veth peer name veth-b

\[root@network ~\]\#

\# 设置虚拟网卡地址

\[root@network ~\]\# ip addr add 10.0.0.1/24 dev veth-a

\[root@network ~\]\# ip link set dev veth-a up

\[root@network ~\]\# ip link set veth-b netns nstest

\[root@network ~\]\# ip netns exec nstest ip addr add 10.0.0.2/24 dev veth-b

\[root@network ~\]\# ip netns exec nstest ip link set dev veth-b up

\# 测试连通性

\[root@network ~\]\# ping 10.0.0.2 -I veth-a

PING 10.0.0.2 \(10.0.0.2\) from 10.0.0.1 veth-a: 56\(84\) bytes of data.

64 bytes from 10.0.0.2: icmp\_seq=1 ttl=64 time=0.048 ms

64 bytes from 10.0.0.2: icmp\_seq=2 ttl=64 time=0.043 ms

64 bytes from 10.0.0.2: icmp\_seq=3 ttl=64 time=0.043 ms

^C

--- 10.0.0.2 ping statistics ---

3 packets transmitted, 3 received, 0% packet loss, time 1999ms

rtt min/avg/max/mdev = 0.043/0.044/0.048/0.008 ms

\[root@network ~\]\#

\[root@network ~\]\#

\[root@network ~\]\# ip netns exec nstest ping 10.0.0.1

PING 10.0.0.1 \(10.0.0.1\) 56\(84\) bytes of data.

64 bytes from 10.0.0.1: icmp\_seq=1 ttl=64 time=0.033 ms

64 bytes from 10.0.0.1: icmp\_seq=2 ttl=64 time=0.042 ms

64 bytes from 10.0.0.1: icmp\_seq=3 ttl=64 time=0.044 ms

