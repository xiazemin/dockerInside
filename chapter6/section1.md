# 层

$docker run docker/whalesay cowsay boo

Unable to find image 'docker/whalesay:latest' locally

latest: Pulling from docker/whalesay

e190868d63f8: Downloading 11.35 MB/65.77 MB

909cd34c6fd7: Download complete

e190868d63f8: Pull complete

909cd34c6fd7: Pull complete

0b9bfabab7c1: Pull complete

其中e190868d63f8 标示层（layer）的ID，docker镜像是在基础镜像基础上，每一次改动提交都是一个层



