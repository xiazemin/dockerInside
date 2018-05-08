# 安装ping

~\# ping wwwbaidu.com

-bash: ping: command not found

~\# apt-get update

Hit:1 [http://archive.ubuntu.com/ubuntu](http://archive.ubuntu.com/ubuntu) xenial InRelease

~\# apt-get install iputils-ping

Reading package lists... Done

Building dependency tree

~\# ping ww.baidu.com

PING ps\_other.a.shifen.com \(123.125.114.144\) 56\(84\) bytes of data.

64 bytes from 123.125.114.144: icmp\_seq=1 ttl=37 time=0.293 ms

\#安装ifconfig

~\# ifconfig

-bash: ifconfig: command not found

~\# apt install net-tools

Reading package lists... Done

~\# ifconfig

eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:08

```
      inet addr:172.17.0.8  Bcast:0.0.0.0  Mask:255.255.0.0

      inet6 addr: fe80::42:acff:fe11:8/64 Scope:Link

      UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

      RX packets:6064 errors:0 dropped:0 overruns:0 frame:0

      TX packets:4381 errors:0 dropped:0 overruns:0 carrier:0

      collisions:0 txqueuelen:0

      RX bytes:6421225 \(6.4 MB\)  TX bytes:343978 \(343.9 KB\)
```

lo        Link encap:Local Loopback

```
      inet addr:127.0.0.1  Mask:255.0.0.0

      inet6 addr: ::1/128 Scope:Host

      UP LOOPBACK RUNNING  MTU:65536  Metric:1

      RX packets:0 errors:0 dropped:0 overruns:0 frame:0

      TX packets:0 errors:0 dropped:0 overruns:0 carrier:0

      collisions:0 txqueuelen:1

      RX bytes:0 \(0.0 B\)  TX bytes:0 \(0.0 B\)
```

宿主机：

~$ifconfig

lo0: flags=8049&lt;UP,LOOPBACK,RUNNING,MULTICAST&gt; mtu 16384

```
options=3&lt;RXCSUM,TXCSUM&gt;

inet6 ::1 prefixlen 128

inet 127.0.0.1 netmask 0xff000000

inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1

nd6 options=1&lt;PERFORMNUD&gt;
```

gif0: flags=8010&lt;POINTOPOINT,MULTICAST&gt; mtu 1280

stf0: flags=0&lt;&gt; mtu 1280

en0: flags=8863&lt;UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST&gt; mtu 1500

```
ether f4:5c:89:c0:7b:d3

inet6 fe80::f65c:89ff:fec0:7bd3%en0 prefixlen 64 scopeid 0x4

inet 172.24.30.69 netmask 0xfffffe00 broadcast 172.24.31.255

nd6 options=1&lt;PERFORMNUD&gt;

media: autoselect

status: active
```

en1: flags=963&lt;UP,BROADCAST,SMART,RUNNING,PROMISC,SIMPLEX&gt; mtu 1500

```
options=60&lt;TSO4,TSO6&gt;

ether 4a:00:05:a9:f7:c0

media: autoselect &lt;full-duplex&gt;

status: inactive
```

en2: flags=963&lt;UP,BROADCAST,SMART,RUNNING,PROMISC,SIMPLEX&gt; mtu 1500

```
options=60&lt;TSO4,TSO6&gt;

ether 4a:00:05:a9:f7:c1

media: autoselect &lt;full-duplex&gt;

status: inactive
```

bridge0: flags=8863&lt;UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST&gt; mtu 1500

```
options=63&lt;RXCSUM,TXCSUM,TSO4,TSO6&gt;

ether f6:5c:89:0c:64:00

Configuration:

    id 0:0:0:0:0:0 priority 0 hellotime 0 fwddelay 0

    maxage 0 holdcnt 0 proto stp maxaddr 100 timeout 1200

    root id 0:0:0:0:0:0 priority 0 ifcost 0 port 0

    ipfilter disabled flags 0x2

member: en1 flags=3&lt;LEARNING,DISCOVER&gt;

        ifmaxaddr 0 port 5 priority 0 path cost 0

member: en2 flags=3&lt;LEARNING,DISCOVER&gt;

        ifmaxaddr 0 port 6 priority 0 path cost 0

nd6 options=1&lt;PERFORMNUD&gt;

media: &lt;unknown type&gt;

status: inactive
```

p2p0: flags=8843&lt;UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST&gt; mtu 2304

```
ether 06:5c:89:c0:7b:d3

media: autoselect

status: inactive
```

awdl0: flags=8943&lt;UP,BROADCAST,RUNNING,PROMISC,SIMPLEX,MULTICAST&gt; mtu 1484

```
ether da:03:ca:8c:92:4a

inet6 fe80::d803:caff:fe8c:924a%awdl0 prefixlen 64 scopeid 0xa

nd6 options=1&lt;PERFORMNUD&gt;

media: autoselect

status: active
```

en4: flags=8863&lt;UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST&gt; mtu 1500

```
options=10b&lt;RXCSUM,TXCSUM,VLAN\_HWTAGGING,AV&gt;

ether 38:c9:86:3b:66:0f

inet6 fe80::3ac9:86ff:fe3b:660f%en4 prefixlen 64 scopeid 0x7

inet 172.24.20.7 netmask 0xfffffc00 broadcast 172.24.23.255

nd6 options=1&lt;PERFORMNUD&gt;

media: autoselect \(1000baseT &lt;full-duplex,energy-efficient-ethernet&gt;\)

status: active
```



宿主机

inet 172.24.20.7 netmask 0xfffffc00 broadcast 172.24.23.255

docker机

inet addr:172.17.0.8 Bcast:0.0.0.0 Mask:255.255.0.0

在不同网段所以直接ssh docker 的ip无法登陆



