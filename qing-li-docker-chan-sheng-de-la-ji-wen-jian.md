首先使用最简单的方法，清理当前未运行的所有Docker容器：

docker system prune

运行结果如下：

davidhopper@davidhopper-ThinkPad-P50s:~/code/apollo$ docker system prune

WARNING! This will remove:

* all stopped containers

* all volumes not used by at least one container

* all networks not used by at least one container

* all dangling images

Are you sure you want to continue? \[y/N\] y

Deleted Volumes:

333739e346364a7d515cdfc585f5231dd7f74a9e71431d152f6efc4da3bcb303

419184335b3c6130cb47b98c672e9666479ef0ff27cd2d20e173b55800507052

