# docker配置独立桥接IP



使用端口映射（NAT）的方式存在一个弊端，当多个容器都需要使用某个端口时或者host主机端口与容器端口冲突时（例如，host主机搭建了80的服务，两个容器也都搭建了80的服务，那个只有1个服务可以使用本机的80端口，其他服务都要映射为其他端口）



 



为容器配置独立的桥接IP就完美的解决了这个问题。以下为配置步骤：



可以查看默认的docker0网卡的IP：







可以看到，是一个虚拟的IP地址172.17.42.1。



 



接下来开始配置，首先停止docker服务：



/etc/init.d/docker stop



 



接着停止docker0网卡：



ifconfig docker0 down



 



删除默认的桥接网络docker0：



brctl delbr docker0



 



创建桥接网卡，修改默认的eth0的配置文件：



cd /etc/sysconfig/network-scripts/修改默认ifcfg-eth0配置文件：







创建一个新的文件ifcfg-br0并编辑：



DEVICE=br0



ONBOOT=yes



NM\_CONTROLLED=no



BOOTPROTO=static



TYPE=Bridge



IPADDR=10.0.0.36



NETMASK=255.255.255.224



GATEWAY=10.0.0.33



上面高亮的3行要根据本机的虚拟网卡信息填写，我本机的信息：











完成后，保存退出，并重启网络服务（service network restart）。



 



可以看到桥接网络已经启动了：







 



修改docker的配置文件/etc/sysconfig/docker，添加桥接网卡参数：







修改完成后重启docker服务：



service restart docker



接着我们启动一个容器：



docker run --name centostest centos:latest /bin/bash



容器启动后可以看到对应的网卡：







至此，就已经配置好桥接网络了

