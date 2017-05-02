# section1
  
5\)docker build 使用此配置生成新的image    
build命令可以从Dockerfile和上下文来创建镜像：   
docker build \[OPTIONS\] PATH \| URL \| -    上面的PATH或URL中的文件被称作上下文，build image的过程会先把这些文件传送到docker的服务端来进行的。    如果PATH直接就是一个单独的Dockerfile文件则可以不需要上下文；如果URL是一个Git仓库地址，那么创建image的过程中会自动git clone一份到本机的临时目录，它就成为了本次build的上下文。无论指定的PATH是什么，Dockerfile是至关重要的，请参考Dockerfile Reference。    请看下面的例子：      
$ cat Dockerfile     
FROM seanlook/nginx:bash\_vim    EXPOSE 80    ENTRYPOINT /usr/sbin/nginx -c /etc/nginx/nginx.conf && /bin/bash       
$ docker build -t seanlook/nginx:bash\_vim\_Df .    
Sending build context to Docker daemon 73.45 MB   
Sending build context to Docker daemon     
Step 0 : FROM seanlook/nginx:bash\_vim     ---> aa8516fa0bb7    
Step 1 : EXPOSE 80     ---> Using cache     ---> fece07e2b515    
Step 2 : ENTRYPOINT /usr/sbin/nginx -c /etc/nginx/nginx.conf && /bin/bash     ---> Running in e08963fd5afb     ---> d9bbd13f5066    
Removing intermediate container e08963fd5afb    
Successfully built d9bbd13f5066    
上面的PATH为.，所以在当前目录下的所有文件 (不包括.dockerignore中的）将会被tar打包并传送到docker daemon (一般在本机），从输出我们可以到Sending build context...，最后有个Removing intermediate container的过程，可以通过--rm=false来保留容器。    TO-DO    docker build github.com/creack/docker-firefox失败。      

6 给镜像打上标签 \(tag\)tag的作用主要有两点：一是为镜像起一个容易理解的名字，二是可以通过docker tag来重新指定镜像的仓库，这样在push时自动提交到仓库。       
将同一IMAGE\_ID的所有tag，合并为一个新的   
$ docker tag 195eb90b5349 seanlook/ubuntu:rm\_test        
新建一个tag，保留旧的那条记录   
$ docker tag Registry/Repos:Tag New\_Registry/New\_Repos:New\_Tag   

7 查看容器的信息container \(ps\)  
docker ps命令可以查看容器的CONTAINER ID、NAME、IMAGE NAME、端口开启及绑定、容器启动后执行的COMMNAD。经常通过ps来找到CONTAINER\_ID。  
docker ps 默认显示当前正在运行中的container   
docker ps -a 查看包括已经停止的所有容器    
docker ps -l 显示最新启动的一个容器 (包括已停止的）        

8 查看容器中正在运行的进程\(top\)   
容器运行时不一定有/bin/bash终端来交互执行top命令，查看container中正在运行的进程，况且还不一定有top命令，这是docker top <\container\_id/container\_name>就很有用了。实际上在host上使用ps -ef\|grep docker也可以看到一组类似的进程信息，把container里的进程看成是host上启动docker的子进程就对了。        

9 其他命令    docker还有一些如login、cp、logs、export、import、load、kill等不是很常用的命令，比较简单，请参考官网。    参考    Official Command Line Reference    docker中文指南cli-widuu翻译    Docker —— 从入门到实践    Docker基础与高级 