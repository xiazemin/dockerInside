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

          inet addr:172.17.0.8  Bcast:0.0.0.0  Mask:255.255.0.0

          inet6 addr: fe80::42:acff:fe11:8/64 Scope:Link

          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

          RX packets:6064 errors:0 dropped:0 overruns:0 frame:0

          TX packets:4381 errors:0 dropped:0 overruns:0 carrier:0

          collisions:0 txqueuelen:0

          RX bytes:6421225 \(6.4 MB\)  TX bytes:343978 \(343.9 KB\)



lo        Link encap:Local Loopback

          inet addr:127.0.0.1  Mask:255.0.0.0

          inet6 addr: ::1/128 Scope:Host

          UP LOOPBACK RUNNING  MTU:65536  Metric:1

          RX packets:0 errors:0 dropped:0 overruns:0 frame:0

          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0

          collisions:0 txqueuelen:1

          RX bytes:0 \(0.0 B\)  TX bytes:0 \(0.0 B\)



