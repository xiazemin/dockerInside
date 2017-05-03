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



