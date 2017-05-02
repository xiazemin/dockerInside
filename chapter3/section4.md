# Docker基本命令使用详解

1. 运行容器

$docker run -i -t ubuntu /bin/bash

-i 标志保证容器中的STDIN是开启的

-t 标志告诉Docker为要创建的容器分配一个伪tty终端

ubuntu 表示我们创建容器使用的镜像

/bin/bash 表示当容器创建完成之后，Docker就会执行容器中的/bin/bash命令2

2.给容器命名

$docker run --name my\_container  -i -t ubuntu /bin/bash

--name为容器指定一个名称，使用指定的容器名称比使用容器ID更方便。

3.重新启动已停止的容器

\#使用容器ID启动容器

$docker start f5a9f05f4214

\#使用容器名称启动容器

$ docker start my\_container

$ docker restart my\_container

除了容器ID，我们还可以使用容器名称来运行容器，也可以用\`docker restart\`命令来重新启动一个容器，运行以上命令，使用\`docker ps\`就可以看到我们的容器已经开始运行了。



