1.停止所有的container，这样才能够删除其中的images：

docker stop $\(docker ps -a -q\)

如果想要删除所有container的话再加一个指令：

docker rm $\(docker ps -a -q\)

2.查看当前有些什么images

docker images

3.删除images，通过image的id来指定删除谁

docker rmi &lt;image id&gt;

想要删除untagged images，也就是那些id为&lt;None&gt;的image的话可以用

docker rmi $\(docker images \| grep "^&lt;none&gt;" \| awk "{print $3}"\)

要删除全部image的话

docker rmi $\(docker images -q\)

$docker rm 6b362a9f73eb

Error response from daemon: No such container: 6b362a9f73eb

原因：删除镜像用的命令是rmi 不是rm

$docker rmi 9772288ea0c5

Error response from daemon: conflict: unable to delete 9772288ea0c5 \(must be forced\) - image is being used by stopped container 854018a04c25

必须先停止容器再删除镜像

$docker ps

$docker stop 2a4a720bd418

$sudo docker rmi 6b362a9f73eb

Error response from daemon: conflict: unable to delete 6b362a9f73eb \(cannot be forced\) - image has dependent child images

```
$ docker image inspect --format='{{.RepoTags}} {{.Id}} {{.Parent}}' $(docker image ls -q --filter since=6b362a9f73eb)
[ifconfig/curl:latest] sha256:56a5eb941d86dd20a942b1eac4faafbb731d269266ce6317dedcc5b98ed55e3d sha256:0f2d38480c888a4df93014c021d91b548e41113467bf45389658d097f93ec198
```

原因是有另外的 image FROM 了这个 image，可以使用下面的命令列出所有在指定 image 之后创建的 image 的父 image

$ docker rmi 36b1e23becabc0b27c5787712dce019982c048665fd9e7e6cb032a46bcac510d

Error response from daemon: conflict: unable to delete 36b1e23becab \(must be forced\) - image is being used by stopped container cb6c7ab61a94

$docker rm cb6c7ab61a94

cb6c7ab61a94

$ docker rmi 36b1e23becabc0b27c5787712dce019982c048665fd9e7e6cb032a46bcac510d

Untagged: hub.c.163.com/library/swarm:latest

$docker rmi 4a415e366388

Error response from daemon: conflict: unable to delete 4a415e366388 \(must be forced\) - image is referenced in multiple repositories

镜像有1个repo引用

root@souyunku:~/mydocker\# docker rmi 4acError response from daemon: conflict: unable to delete 4ac2d12f10cd \(must be forced\) - image is referencedinmultiple repositories

**2.解决方法**

**删除REPOSITORY**

被删除的ImageID，这里存在1个REPOSITORY名字引用，解决方法如下：

即删除时指定名称，而不是IMAGE ID。

root@souyunku:~/mydocker\# docker rmi souyunku/nginx:v1Untagged: souyunku/nginx:v1

再删除IMAGE ID就可以了：



