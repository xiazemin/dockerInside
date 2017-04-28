# 打tag上传镜像

$ docker tag 8e15421920b1 xiazemin/docker-whale:latest

Error response from daemon: no such id: 8e15421920b1

 $ docker tag ca5555513a01 xiazemin/docker-whale:latest

 $ docker push xiazemin/docker-whale

The push refers to a repository \[docker.io/xiazemin/docker-whale\]

dd1361ddca89: Pushing 7.301 MB/29.64 MB

5f70bf18a086: Mounted from docker/whalesay

d061ee1340ec: Mounted from docker/whalesay

d511ed9e12e1: Mounted from docker/whalesay

091abc5148e4: Mounted from docker/whalesay

b26122d57afa: Mounted from docker/whalesay

37ee47034d9b: Mounted from docker/whalesay

528c8710fd95: Mounted from docker/whalesay

1154ba695078: Mounted from docker/whalesay