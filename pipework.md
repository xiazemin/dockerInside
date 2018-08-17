下面我们来操作一下，我主机 A 地址为 192.168.157.128/24,网关为 192.168.157.2,需要给 Docker容器的地址配置为 192.168.157.150/24。在主机 A 上做如下操作：



安装 pipework



下载地址：wgethttps://github.com/jpetazzo/pipework.git



unzippipework-master.zip



cp -p /root/pipework-master/pipework/usr/local/bin/



启动 Docker 容器。



docker run -itd--name test1 镜像 /bin/bash



配置容器网络，并连到网桥 br0 上。网关在 IP 地址后面加@指定。



pipework br0 test1192.168.157.150/24@192.168.157.2



将主机 eno16777736 桥接到 br0 上，并把 eno16777736 的 IP 配置在 br0 上。



p addradd 192.168.157.128/24 dev br0



ip addrdel 192.168. 157.128/24 dev eno16777736



brctladdif br0 eno16777736



ip routedel default



ip routeadd default via 192.168.157.2 dev br0



注：如果是远程操作，中间网络会断掉，所以放在一条命令中执行。



ip addradd 192.168.157.128/24 dev br0；ip addr del 192.168. 157.128/24 deveno16777736；brctl addif br0 eno16777736；ip route del default；ip route add default via 192.168.157.2dev br0



完成上述步骤后，我们发现 Docker 容器已经可以使用新的 IP 和主机网络里的机器相互通信了

