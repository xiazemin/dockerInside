# 安装docker

1，[Boot2Docker](https://github.com/boot2docker/boot2docker)（官方已废弃）

        Docker引擎使用了Linux内核特定的特性，所以要让它运行在OS X上我们需要用一个轻量型的虚拟机\(vm\)。用OS X的Docker客户端来控制虚拟Docker来构建，运行以及管理Docker容器。为了使过程更简单一点，设计了一个叫做[Boot2Docker](https://github.com/boot2docker/boot2docker)的帮助应用程序，它能按照虚拟机以及运行Docker后台程序。

     $boot2docker init

     $ boot2docker start

     $ export DOCKER\_HOST=tcp://$\(boot2docker ip 2&gt;/dev/null\):2375

完成虚拟化环境搭建。

     

2，docker-for-mac

  下载地址：https://docs.docker.com/docker-for-mac/





