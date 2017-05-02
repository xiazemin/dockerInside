# docker常用命令
总的来说分为以下几种：  
容器生命周期管理 — docker \[run\|start\|stop\|restart\|kill\|rm\|pause\|unpause\]  
容器操作运维 — docker \[ps\|inspect\|top\|attach\|events\|logs\|wait\|export\|port\]  
容器rootfs命令 — docker \[commit\|cp\|diff\]  
镜像仓库 — docker \[login\|pull\|push\|search\]  
本地镜像管理 — docker \[images\|rmi\|tag\|build\|history\|save\|import\]  
其他命令 — docker \[info\|version\]    

1 列出机器上的镜像 \(images\)  
$ docker images   
REPOSITORY               TAG             IMAGE ID        CREATED         VIRTUAL SIZE  ubuntu                   14.10           2185fd50e2ca    13 days ago     236.9 MB  其中我们可以根据REPOSITORY来判断这个镜像是来自哪个服务器，如果没有 / 则表示官方镜像，类似于username/repos\_name表示Github的个人公共库，类似于regsistory.example.com:5000/repos\_name则表示的是私服。  IMAGE ID列其实是缩写，要显示完整则带上--no-trunc选项    

2 在docker index中搜索image (search）  
Usage: docker search TERM  
$ docker search seanlo  NAME                
DESCRIPTION           STARS     OFFICIAL   AUTOMATED  seanloook/centos6   sean's docker repos         0  搜索的范围是官方镜像和所有个人公共镜像。NAME列的 / 后面是仓库的名字。  

3 从docker registry server 中下拉image或repository (pull）  
Usage: docker pull \[OPTIONS\] NAME\[:TAG\]  
$ docker pull centos  
上面的命令需要注意，在docker v1.2版本以前，会下载官方镜像的centos仓库里的所有镜像，而从v.13开始官方文档里的说明变了：will pull the centos:latest image, its intermediate layers and any aliases of the same id，也就是只会下载tag为latest的镜像 (以及同一images id的其他tag）。  也可以明确指定具体的镜像：  
$ docker pull centos:centos6  当然也可以从某个人的公共仓库 (包括自己是私人仓库）拉取，形如docker pull username/repository <:tag\_name>  
$ docker pull seanlook/centos:centos6  如果你没有网络，或者从其他私服获取镜像，形如
docker pull registry.domain.com:5000/repos:<tag\_name>  
$ docker pull dl.dockerpool.com:5000/mongo:latest    

4 推送一个image或repository到registry (push）  
与上面的pull对应，可以推送到Docker Hub的Public、Private以及私服，但不能推送到Top Level Repository。  
$ docker push seanlook/mongo 
$ docker push registry.tp-link.net:5000/mongo:2014-10-27  registry.tp-link.NET也可以写成IP，172.29.88.222。  在repository不存在的情况下，命令行下push上去的会为我们创建为私有库，然而通过浏览器创建的默认为公共库。    

5 从image启动一个container (run）    
docker run命令首先会从特定的image创之上create一层可写的Container，然后通过start命令来启动它。停止的container可以重新启动并保留原来的修改。run命令启动参数有很多，以下是一些常规使用说明，更多部分请参考http://www.cnphp6.com/archives/24899    当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：        检查本地是否存在指定的镜像，不存在就从公有仓库下载    利用镜像创建并启动一个容器    分配一个文件系统，并在只读的镜像层外面挂载一层可读写层    从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去    从地址池配置一个 ip 地址给容器    执行用户指定的应用程序    执行完毕后容器被终止    
Usage: docker run \[OPTIONS\] IMAGE \[COMMAND\] \[ARG...\]  