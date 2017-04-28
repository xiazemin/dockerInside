# 生成镜像文件
注意不要少了最后一个参数，表示目录

 $ docker build -t docker-whale

"docker build" requires exactly 1 argument\(s\).

See 'docker build --help'.

Usage:  docker build \[OPTIONS\] PATH \| URL \| -

Build an image from a Dockerfile
正确（但是源有问题）
 $ docker build -t docker-whale .

Sending build context to Docker daemon 2.048 kB

Step 1/3 : FROM docker/whalesay:latest

 ---&gt; 6b362a9f73eb

Step 2/3 : RUN apt-get -y update && apt-get install -y fortunes

 ---&gt; Running in 4abd882905f9

Ign http://archive.ubuntu.com trusty InRelease

Get:1 http://archive.ubuntu.com trusty-updates InRelease \[65.9 kB\]

Get:2 http://archive.ubuntu.com trusty-security InRelease \[65.9 kB\]

Hit http://archive.ubuntu.com trusty Release.gpg

Hit http://archive.ubuntu.com trusty Release

Get:3 http://archive.ubuntu.com trusty-updates/main Sources \[489 kB\]

Get:4 http://archive.ubuntu.com trusty-updates/main amd64 Packages \[522 kB\]

Get:5 http://archive.ubuntu.com trusty-updates/restricted amd64 Packages \[522 kB\]

Get:6 http://archive.ubuntu.com trusty-updates/restricted Sources \[489 kB\]

Get:7 http://archive.ubuntu.com trusty-updates/universe amd64 Packages \[522 kB\]

Get:8 http://archive.ubuntu.com trusty-security/main amd64 Packages \[522 kB\]

Get:9 http://archive.ubuntu.com trusty-updates/universe Sources \[489 kB\]

Get:10 http://archive.ubuntu.com trusty-security/restricted amd64 Packages \[522 kB\]

Get:11 http://archive.ubuntu.com trusty-security/universe amd64 Packages \[522 kB\]

Get:12 http://archive.ubuntu.com trusty-security/main Sources \[489 kB\]

Get:13 http://archive.ubuntu.com trusty/main amd64 Packages \[522 kB\]

Get:14 http://archive.ubuntu.com trusty/restricted amd64 Packages \[522 kB\]

Get:15 http://archive.ubuntu.com trusty-security/restricted Sources \[489 kB\]

Get:16 http://archive.ubuntu.com trusty/universe amd64 Packages \[522 kB\]

Get:17 http://archive.ubuntu.com trusty-security/universe Sources \[489 kB\]

Get:18 http://archive.ubuntu.com trusty/main Sources \[489 kB\]

Get:19 http://archive.ubuntu.com trusty/restricted Sources \[489 kB\]

Get:20 http://archive.ubuntu.com trusty/universe Sources \[489 kB\]

W: Size of file /var/lib/apt/lists/archive.ubuntu.com\_ubuntu\_dists\_trusty\_main\_binary-amd64\_Packages.gz is not what the server reported 522218 1743009

Fetched 9228 kB in 14s \(643 kB/s\)

W: Size of file /var/lib/apt/lists/archive.ubuntu.com\_ubuntu\_dists\_trusty\_main\_source\_Sources.gz is not what the server reported 488526 1334581

W: Size of file /var/lib/apt/lists/archive.ubuntu.com\_ubuntu\_dists\_trusty\_universe\_source\_Sources.gz is not what the server reported 488526 7925687

W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-updates/main/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-updates/restricted/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-updates/universe/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-updates/main/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-updates/restricted/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-updates/universe/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-security/main/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-security/restricted/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-security/universe/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-security/main/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-security/restricted/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-security/universe/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/main/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/restricted/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/universe/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/main/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/restricted/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/universe/binary-amd64/Packages  Hash Sum mismatch



E: Some index files failed to download. They have been ignored, or old ones used instead.

The command '/bin/sh -c apt-get -y update && apt-get install -y fortunes' returned a non-zero code: 100


