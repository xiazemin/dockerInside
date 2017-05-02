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