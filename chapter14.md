# 查看网络

$docker network ls

NETWORK ID          NAME                DRIVER              SCOPE

5c50c967bbfd        bridge              bridge              local

34033bd99114        docker0             bridge              local

8edecb0e4997        host                host                local

e19d371e8b77        none                null                local

$docker network inspect bridge

$ifconfig docker0 172.17.0.1

ifconfig: interface docker0 does not exist

$docker network create ....

Docker for Mac

$ docker --version

Docker version 1.13.0, build 49bf474



$ docker-compose --version

docker-compose version 1.10.0, build 4bd6f1a



$ docker-machine --version

docker-machine version 0.9.0, build 15fd4c7

