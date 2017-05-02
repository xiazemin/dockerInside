# Docker管理工具

Docker管理工具Web UI：DockerUI & Shipyard

本文主要介绍两款Docker Web管理工具：DockerUI及Shipyard，并对它们的部署、功能及使用进行对比。

后续会介绍Docker近日最新发布的容器管理利器：swarm。

部署方面

DockerUI

Run cmd docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock dockerui/dockerui

Open your browser to [http://&lt;dockerd](http://<dockerd) host ip&gt;:9000

Shipyard

Run cmd docker run --rm -v /var/run/docker.sock:/var/run/docker.sock shipyard/deploy start

Open your browser to [http://&lt;dockerd](http://<dockerd) host ip&gt;:8080, username: admin, password: shipyard

DockerUI部署很顺利，没遇到任何问题。

Shipyard实际使用过程中遇到一些问题，如：iptables问题。

功能及使用体验方面

两者各有优缺点，比较适合配合使用。

DockerUI

DockerUI基于Docker API，提供等同Docker命令行的大部分功能，支持container管理，image管理。

优点：

支持container批量操作；

支持image管理（虽然比较薄弱）

缺点：

不支持多主机。

Shipyard

Shipyard也是完全基于Docker API，支持container管理、engine管理（一个engine就是监听tcp端口的docker daemon）。

优点：

支持多主机；

支持container及engine资源限制及图形展示；

支持container实例横向扩展；

支持批量创建；

支持创建时自动调度。

缺点：

不支持image管理；

不支持container批量操作。

docker pull hub.c.163.com/longjuxu/shipyard/shipyard:latest



