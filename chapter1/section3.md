# Dockerfile

Dockerfile用来创建一个自定义的image,包含了用户指定的软件依赖等。

当前目录下包含Dockerfile,使用命令build来创建新的image

$ cat Dockerfile

FROM docker/whalesay:latest

RUN apt-get -y update && apt-get install -y fortunes

CMD /usr/games/fortune -a \| cowsay



