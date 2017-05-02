# 存储和载入镜像

存储镜像
如果要导出镜像到本地文件，可以使用 docker save 命令。
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              14.04               c4ff7513909d        5 weeks ago         225.4 MB
...
$sudo docker save -o ubuntu\_14.04.tar ubuntu:14.04

载入镜像
可以使用 docker load 从导出的本地文件中再导入到本地镜像库，例如
$ sudo docker load --input ubuntu\_14.04.tar
或
$ sudo docker load ubuntu\_14.04.tar
这将导入镜像以及其相关的元数据信息（包括标签等）。

Docker exec与Docker attach
不论是开发者是运维人员，都经常有需要进入容器的诉求。
目前主要的方法有以下几种：
1. 使用ssh登陆进容器
2. 使用nsenter、nsinit等第三方工具
3. 使用docker本身提供的工具

方法1需要在容器中启动sshd，存在开销和攻击面增大的问题。同时也违反了Docker所倡导
的一个容器一个进程的原则。
方法2需要额外学习使用第三方工具。
所以大多数情况最好还是使用Docker原生方法，Docker目前主要提供了Docker exec和
Docker attach两个命令。
Docker attach
Docker attach可以attach到一个已经运行的容器的stdin，然后进行命令执行的动作。
但是需要注意的是，如果从这个stdin中exit，会导致容器的停止。
Docker exec
docker exec -it containerId /bin/sh
不会像attach方式因为退出，导致整个容器退出。
这种方式可以替代ssh或者nsenter、nsinit方式，在容器内进行操作。

docker inspect
docker inspect -f {{.IPAddress}} 来获取容器的 IP 地址 “.” 表示“当前上下文”。
更多请参见http://www.tuicool.com/articles/V3AvI3I

docker run相关参数
Clean up \(--rm\)
默认情况下，每个container在退出时，它的文件系统也会保存下来。这样一方面调试会方便些，因为你可以通过查看日志等方式来确定最终状态。另外一方面，你也可以保存container所产生的数据。但是当你仅仅需要短期的运行一个前台container，这些数据同时不需要保留时。你可能就希望docker能在container结束时自动清理其所产生的数据。
这个时候你就需要--rm这个参数了。 注意：--rm 和 -d不能共用！ 
CMD \(default command or options\)
$ sudo docker run \[OPTIONS\] IMAGE\[:TAG\] \[COMMAND\] \[ARG...\]
这条命令中的COMMAND部分是可选的。因为这个IMAGE在build时，开发人员可能已经设定了默认执行的command。作为操作人员，你可以使用上面命令中新的command来覆盖旧的command。
如果image中设定了ENTRYPOINT，那么命令中的CMD也可以作为参数追加到ENTRYPOINT中。
ENTRYPOINT \(default command to execute at runtime\)
--entrypoint="": Overwrite the default entrypoint set by the image
这个ENTRYPOINT和COMMAND类似，它指定了当container执行时，需要启动哪些进程。相对COMMAND而言，ENTRYPOINT是比较困难进行覆盖的，这个ENTRYPOINT可以让container设定默认启动行为，所以当container启动时，你可以执行任何一个二进制可执行程序。你也可以通过COMMAND给这个ENTRYPOINT传递参数。但当你需要再container中执行其他进程时，你就可以指定其他ENTRYPOINT了。
下面就是一个例子，container可以在启动时自动执行shell，然后启动其它进程。
$ sudo docker run -i -t --entrypoint /bin/bash example/redis
\#or two examples of how to pass more parameters to that ENTRYPOINT:
$ sudo docker run -i -t --entrypoint /bin/bash example/redis -c ls -l
$ sudo docker run -i -t --entrypoint /usr/bin/redis-cli example/redis --help
EXPOSE \(incoming ports\)
Dockefile在网络方面除了提供一个EXPOSE之外，没有提供其它选项。下面这些参数可以覆盖Dockefile的expose默认值：
--expose=\[\]: Expose a port or a range of ports from the container
without publishing it to your host
-P=false   : Publish all exposed ports to the host interfaces
-p=\[\]      : Publish a container᾿s port to the host \(format:
ip:hostPort:containerPort \| ip::containerPort \|
hostPort:containerPort \| containerPort\)
\(use 'docker port' to see the actual mapping\)
--link=""  : Add link to another container \(name:alias\)
--expose可以让container接受外部传入的数据。container内监听的port不需要和外部host的port相同。比如说在container内部，一个HTTP服务监听在80端口，对应外部host的port就可能是49880.
操作人员可以使用--expose，让新的container访问到这个container。具体有三个方式：
1. 使用-p来启动container。
2. 使用-P来启动container。
3. 使用--link来启动container。
如果使用-p或者-P，那么container会开发部分端口到host，只要对方可以连接到host，就可以连接到container内部。当使用-P时，docker会在host中随机从49153 和65535之间查找一个未被占用的端口绑定到container。你可以使用docker port来查找这个随机绑定端口。
当你使用--link方式时，作为客户端的container可以通过私有网络形式访问到这个container。同时Docker会在客户端的container中设定一些环境变量来记录绑定的IP和PORT。
WORKDIR
container中默认的工作目录是根目录\(/\)。开发人员可以通过Dockerfile的WORKDIR来设定默认工作目录，操作人员可以通过"-w"来覆盖默认的工作目录。
更多请参考http://www.tuicool.com/articles/uUBVJr
可见,以上参数在dockerfile有相关对应.