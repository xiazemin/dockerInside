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

如果空间释放得还够多，就想办法删除/var/lib/docker/aufs文件夹，最朴素的想法是直接删除：



sudo rm -rf /var/lib/docker/aufs



结果令人失望，提示信息如下：



davidhopper@davidhopper-ThinkPad-P50s:~$ sudo rm -rf /var/lib/docker/aufs

\[sudo\] davidhopper 的密码： 

rm: 无法删除'/var/lib/docker/aufs': 设备或资源忙



用命令cat /proc/mounts \| grep "docker"查找设备加载情况，果然有aufs：



davidhopper@davidhopper-ThinkPad-P50s:~$ cat /proc/mounts \| grep "docker"

/dev/sda6 /var/lib/docker/aufs ext4 rw,relatime,errors=remount-ro,data=ordered 0 0

没办法，逼我用绝招，先缷载设备，再删除之：



sudo umount /var/lib/docker/aufs

sudo rm -rf /var/lib/docker/aufs

