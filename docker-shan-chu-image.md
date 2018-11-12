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

