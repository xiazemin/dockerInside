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

4.附着到容器上

$docker attach my\_container

Docker容器重新启动的时候，会沿用\`docker run\`命令时制定的参数来运行，因此我们的容器重新启动后会运行一个交互式的shell，此外可以用\`docker attach\`命令重新附着到该容器的会话上。运行命令之后可以需要按下回车键才能进入该会话，如果退出容器的shell，容器会再次停止运行。

1. 创建守护式容器

$docker run --name my\_container -d ubuntu /bin/bash

-d 标志Docker会将容器放到后台运行

\`docker exec\`命令会在容器内部额外启动新进程，可以在容器内运行的进程有两种类型：后台任务和交互式任务。

\#在容器中运行后台任务

$ sudo docker exec -d my\_container touch /etc/new\_config\_file

\#在容器内运行交互式任务

$ sudo docker exec -t -i my\_container /bin/bash

6.停止守护式容器

\#通过容器名称停止正在运行的容器

$ sudo docker stop my\_container

\#通过容器ID停止正在运行的容器

$ sudo docker stop f5a9f05f4214

\#停止容器进程

$ sudo docker kill f5a9f05f4214

如果想快速停止某个容器，使用\`docker kill\`命令在向容器发送停止信号。

7.自动重启容器

$ sudo docker run --restart=always --name my\_container -d ubuntu /bin/bash

--restart 标志会检查容器的退出代码，并据此来决定是否要重启容器，默认是不会重启。

--restart的参数说明

always：无论容器的退出代码是什么，Docker都会自动重启该容器。

on-failure：只有当容器的退出代码为非0值的时候才会自动重启。另外，该参数还接受一个可选的重启次数参数，\`--restart=on-fialure:5\`表示当容器退出代码为非0时，Docker会尝试自动重启该容器，最多5次。

