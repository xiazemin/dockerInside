# DockerUI使用

$ docker pull registry.cn-hangzhou.aliyuncs.com/infinitus/dockerui

Using default tag: latest

latest: Pulling from infinitus/dockerui

841194d080c8: Pull complete

Digest: sha256:4a590b60ced01f04c10bd27aa1b0029bd8f7d00c94becc3522ea40bb8064d765

Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/infinitus/dockerui:latest

$ docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock registry.cn-hangzhou.aliyuncs.com/infinitus/dockerui

463b6e26bae471725bafd95a0db6a443356dc226275a0b71e1172967fea7cf4c

[http://127.0.0.1:9000/\#/](http://127.0.0.1:9000/#/)

![](/assets/importui.png)1）Dashboard控制台。点击Running Containers下面活跃的容器，进入容器的管理界面进行相关操作，比如修改容器名，commit提交容器为新的镜像等。

![](/assets/importui4.png)

![](/assets/importui2.png)![](/assets/importui3.png)

2）container容器管理。点击Display All ，可以显示所有创建了的容器，包括没有启动的。然后点击Action，可以对容器进行启动，关闭，重启，删除，挂起等操作。

![](/assets/importui5.png)3）images镜像管理。点击Action，可以对已有的镜像镜像移除操作。点击Pull，可以拉取镜像。点击镜像ID进去后可以添加或移除镜像tag

![](/assets/importui6.png)如下截图，Pull镜像的时候，Registry为空，默认从docker hub上拉取镜像。

![](/assets/importui7.png)

点击镜像ID进入，可以添加或删除镜像tag标识。

![](/assets/importui8.png)点击ID进入，可以启动容器

