### 不同主机间容器通信 {#articleHeader32}

~\# apt-get install iputils-arping bridge-utils -y

Reading package lists... Done

~\# brctl show

bridge name    bridge id        STP enabled    interfaces

～$docker run -d -p 222:22  --net=bridge ifconfig/curl  /usr/sbin/sshd -D

e4ae8449047fb12991e12fd85932ae24e0a9a2eab9a46c50852661900b2d824a

默认也是桥接模式

列出所有docker网络

～$docker network ls

NETWORK ID          NAME                DRIVER              SCOPE

77f42967f231        bridge              bridge              local

8edecb0e4997        host                host                local

e19d371e8b77        none                null                local

要检查其属性，运行

～$ docker network inspect bridge

  


