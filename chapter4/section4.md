# DockerUI使用

$ docker pull registry.cn-hangzhou.aliyuncs.com/infinitus/dockerui

Using default tag: latest

latest: Pulling from infinitus/dockerui

841194d080c8: Pull complete

Digest: sha256:4a590b60ced01f04c10bd27aa1b0029bd8f7d00c94becc3522ea40bb8064d765

Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/infinitus/dockerui:latest

$ docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock registry.cn-hangzhou.aliyuncs.com/infinitus/dockerui

463b6e26bae471725bafd95a0db6a443356dc226275a0b71e1172967fea7cf4c



