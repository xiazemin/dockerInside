docker ps -a \| awk '{print $1}'\|xargs docker stop

docker ps -a \| awk '{print $1}'\|xargs docker rm

1. Docker System命令

在《谁用光了磁盘？Docker System命令详解》中，我们详细介绍了Docker System命令，它可以用于管理磁盘空间。



docker system df命令，类似于Linux上的df命令，用于查看Docker的磁盘使用情况：

docker system df

TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE

Images              147                 36                  7.204GB             3.887GB \(53%\)

Containers          37                  10                  104.8MB             102.6MB \(97%\)

Local Volumes       3                   3                   1.421GB             0B \(0%\)

Build Cache                                                 0B                  0B



可知，Docker镜像占用了7.2GB磁盘，Docker容器占用了104.8MB磁盘，Docker数据卷占用了1.4GB磁盘。



docker system prune命令可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像（即无tag的镜像）。docker system prune -a命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了……所以使用之前一定要想清楚吶。



执行docker system prune -a命令之后，Docker占用的磁盘空间减少了很多：

docker system df

TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE

Images              10                  10                  2.271GB             630.7MB \(27%\)

Containers          10                  10                  2.211MB             0B \(0%\)

Local Volumes       3                   3                   1.421GB             0B \(0%\)

Build Cache                                                 0B                  0B



2. 手动清理Docker镜像/容器/数据卷

对于旧版的Docker（版本1.13之前），是没有Docker System命令的，因此需要进行手动清理。这里给出几个常用的命令：



删除所有关闭的容器：

docker ps -a \| grep Exit \| cut -d ' ' -f 1 \| xargs docker rm



删除所有dangling镜像（即无tag的镜像）：

docker rmi $\(docker images \| grep "^&lt;none&gt;" \| awk "{print $3}"\)



删除所有dangling数据卷（即无用的Volume）：

docker volume rm $\(docker volume ls -qf dangling=true\)



3. 限制容器的日志大小

有一次，当我使用1与2提到的方法清理磁盘之后，发现并没有什么作用，于是，我进行了一系列分析。



在Ubuntu上，Docker的所有相关文件，包括镜像、容器等都保存在/var/lib/docker/目录中：

du -hs /var/lib/docker/

97G /var/lib/docker/



Docker竟然使用了将近100GB磁盘，这也是够了。使用du命令继续查看，可以定位到真正占用这么多磁盘的目录：

92G  /var/lib/docker/containers/a376aa694b22ee497f6fc9f7d15d943de91c853284f8f105ff5ad6c7ddae7a53



由docker ps可知，Nginx容器的ID恰好为a376aa694b22，与上面的目录/var/lib/docker/containers/a376aa694b22的前缀一致：

docker ps

CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS              PORTS               NAMES

a376aa694b22        192.168.59.224:5000/nginx:1.12.1            "nginx -g 'daemon off"   9 weeks ago         Up 10 minutes                           nginx



因此，Nginx容器竟然占用了92GB的磁盘。进一步分析可知，真正占用磁盘空间的是Nginx的日志文件。那么这就不难理解了。我们Fundebug每天的数据请求为百万级别，那么日志数据自然非常大。



使用truncate命令，可以将Nginx容器的日志文件“清零”：

truncate -s 0 /var/lib/docker/containers/a376aa694b22ee497f6fc9f7d15d943de91c853284f8f105ff5ad6c7ddae7a53/\*-json.log



当然，这个命令只是临时有作用，日志文件迟早又会涨回来。要从根本上解决问题，需要限制Nginx容器的日志文件大小。这个可以通过配置日志的max-size来实现，下面是Nginx容器的docker-compose配置文件：

nginx:

image: nginx:1.12.1

restart: always

logging:

driver: "json-file"

options:

  max-size: "5g"



重启Nginx容器之后，其日志文件的大小就被限制在5GB，再也不用担心了~







