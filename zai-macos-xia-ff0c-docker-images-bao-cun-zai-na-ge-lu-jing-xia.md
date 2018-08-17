/Users/{YourUserName}/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.qcow2



清理所有停止的容器

docker container prune 

清理所有不用数据\(停止的容器,不使用的volume,不使用的networks,悬挂的镜像\)

docker system prune -a

