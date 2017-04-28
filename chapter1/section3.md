# section2

localhost:docker didi$                 docker build -t docker-whale

"docker build" requires exactly 1 argument\(s\).

See 'docker build --help'.



Usage:  docker build \[OPTIONS\] PATH \| URL \| -



Build an image from a Dockerfile

localhost:docker didi$

localhost:docker didi$

localhost:docker didi$ cat Dockerfile

FROM docker/whalesay:latest

RUN apt-get -y update && apt-get install -y fortunes

CMD /usr/games/fortune -a \|cowsay

localhost:docker didi$ docker build -t docker-whale

"docker build" requires exactly 1 argument\(s\).

See 'docker build --help'.



Usage:  docker build \[OPTIONS\] PATH \| URL \| -



Build an image from a Dockerfile

localhost:docker didi$ docker build -h

Flag shorthand -h has been deprecated, please use --help



Usage:	docker build \[OPTIONS\] PATH \| URL \| -



Build an image from a Dockerfile



Options:

      --build-arg list             Set build-time variables \(default \[\]\)

      --cache-from stringSlice     Images to consider as cache sources

      --cgroup-parent string       Optional parent cgroup for the container

      --compress                   Compress the build context using gzip

      --cpu-period int             Limit the CPU CFS \(Completely Fair Scheduler\) period

      --cpu-quota int              Limit the CPU CFS \(Completely Fair Scheduler\) quota

  -c, --cpu-shares int             CPU shares \(relative weight\)

      --cpuset-cpus string         CPUs in which to allow execution \(0-3, 0,1\)

      --cpuset-mems string         MEMs in which to allow execution \(0-3, 0,1\)

      --disable-content-trust      Skip image verification \(default true\)

  -f, --file string                Name of the Dockerfile \(Default is 'PATH/Dockerfile'\)

      --force-rm                   Always remove intermediate containers

      --help                       Print usage

      --isolation string           Container isolation technology

      --label list                 Set metadata for an image \(default \[\]\)

  -m, --memory string              Memory limit

      --memory-swap string         Swap limit equal to memory plus swap: '-1' to enable unlimited swap

      --network string             Set the networking mode for the RUN instructions during build \(default "default"\)

      --no-cache                   Do not use cache when building the image

      --pull                       Always attempt to pull a newer version of the image

FROM docker/whalesay:latest

  -q, --quiet                      Suppress the build output and print image ID on success

      --rm                         Remove intermediate containers after a successful build \(default true\)

      --security-opt stringSlice   Security options

      --shm-size string            Size of /dev/shm, default value is 64MB

      --squash                     Squash newly built layers into a single new layer

  -t, --tag list                   Name and optionally a tag in the 'name:tag' format \(default \[\]\)

      --ulimit ulimit              Ulimit options \(default \[\]\)

localhost:docker didi$ docker build -t docker-whale

"docker build" requires exactly 1 argument\(s\).

See 'docker build --help'.



Usage:  docker build \[OPTIONS\] PATH \| URL \| -



Build an image from a Dockerfile

localhost:docker didi$ lds

-bash: lds: command not found

localhost:docker didi$ ls

Dockerfile

localhost:docker didi$ vi Dockerfile

localhost:docker didi$

localhost:docker didi$ cat Dockerfile

FROM docker/whalesay:latest

RUN apt-get -y update && apt-get install -y fortunes

CMD /usr/games/fortune -a \|cowsay

localhost:docker didi$ vi Dockerfile

localhost:docker didi$

localhost:docker didi$ docker build -t docker-whale

"docker build" requires exactly 1 argument\(s\).

See 'docker build --help'.



Usage:  docker build \[OPTIONS\] PATH \| URL \| -



Build an image from a Dockerfile

localhost:docker didi$ docker build -h

Flag shorthand -h has been deprecated, please use --help



Usage:	docker build \[OPTIONS\] PATH \| URL \| -



Build an image from a Dockerfile



Options:

      --build-arg list             Set build-time variables \(default \[\]\)

      --cache-from stringSlice     Images to consider as cache sources

      --cgroup-parent string       Optional parent cgroup for the container

      --compress                   Compress the build context using gzip

      --cpu-period int             Limit the CPU CFS \(Completely Fair Scheduler\) period

      --cpu-quota int              Limit the CPU CFS \(Completely Fair Scheduler\) quota

  -c, --cpu-shares int             CPU shares \(relative weight\)

      --cpuset-cpus string         CPUs in which to allow execution \(0-3, 0,1\)

      --cpuset-mems string         MEMs in which to allow execution \(0-3, 0,1\)

      --disable-content-trust      Skip image verification \(default true\)

  -f, --file string                Name of the Dockerfile \(Default is 'PATH/Dockerfile'\)

      --force-rm                   Always remove intermediate containers

      --help                       Print usage

      --isolation string           Container isolation technology

      --label list                 Set metadata for an image \(default \[\]\)

  -m, --memory string              Memory limit

      --memory-swap string         Swap limit equal to memory plus swap: '-1' to enable unlimited swap

      --network string             Set the networking mode for the RUN instructions during build \(default "default"\)

      --no-cache                   Do not use cache when building the image

      --pull                       Always attempt to pull a newer version of the image

  -q, --quiet                      Suppress the build output and print image ID on success

      --rm                         Remove intermediate containers after a successful build \(default true\)

      --security-opt stringSlice   Security options

      --shm-size string            Size of /dev/shm, default value is 64MB

      --squash                     Squash newly built layers into a single new layer

  -t, --tag list                   Name and optionally a tag in the 'name:tag' format \(default \[\]\)

      --ulimit ulimit              Ulimit options \(default \[\]\)

localhost:docker didi$ docker build -t docker-whale .

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

localhost:docker didi$ docker build -t docker-whale .

Sending build context to Docker daemon 2.048 kB

Step 1/3 : FROM docker/whalesay:latest

 ---&gt; 6b362a9f73eb

Step 2/3 : RUN apt-get -y update && apt-get install -y fortunes

 ---&gt; Running in 172b18ae37c0

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

Get:9 http://archive.ubuntu.com trusty-security/restricted amd64 Packages \[522 kB\]

Get:10 http://archive.ubuntu.com trusty-security/universe amd64 Packages \[522 kB\]

Get:11 http://archive.ubuntu.com trusty/main amd64 Packages \[522 kB\]

Get:12 http://archive.ubuntu.com trusty/restricted amd64 Packages \[522 kB\]

Get:13 http://archive.ubuntu.com trusty/universe amd64 Packages \[522 kB\]

Get:14 http://archive.ubuntu.com trusty-updates/universe Sources \[489 kB\]

Get:15 http://archive.ubuntu.com trusty-security/main Sources \[489 kB\]

Get:16 http://archive.ubuntu.com trusty-security/restricted Sources \[489 kB\]

Get:17 http://archive.ubuntu.com trusty-security/universe Sources \[489 kB\]

Get:18 http://archive.ubuntu.com trusty/main Sources \[489 kB\]

Get:19 http://archive.ubuntu.com trusty/restricted Sources \[489 kB\]

Get:20 http://archive.ubuntu.com trusty/universe Sources \[489 kB\]

Fetched 9228 kB in 32s \(282 kB/s\)

W: Size of file /var/lib/apt/lists/archive.ubuntu.com\_ubuntu\_dists\_trusty\_main\_binary-amd64\_Packages.gz is not what the server reported 522218 1743009

W: Size of file /var/lib/apt/lists/archive.ubuntu.com\_ubuntu\_dists\_trusty\_universe\_binary-amd64\_Packages.gz is not what the server reported 522218 7588885

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



\# 这里是注释

W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty-security/universe/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/main/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/restricted/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/universe/source/Sources  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/main/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/restricted/binary-amd64/Packages  Hash Sum mismatch



W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/universe/binary-amd64/Packages  Hash Sum mismatch



E: Some index files failed to download. They have been ignored, or old ones used instead.

The command '/bin/sh -c apt-get -y update && apt-get install -y fortunes' returned a non-zero code: 100

localhost:docker didi$

localhost:docker didi$

localhost:docker didi$

localhost:docker didi$              ls

Dockerfile

localhost:docker didi$ vi Dockerfile

localhost:docker didi$

localhost:docker didi$

localhost:docker didi$ docker build -t docker-whale .

Sending build context to Docker daemon 2.048 kB

Step 1/4 : FROM ubuntu:14.04

14.04: Pulling from library/ubuntu

8f229c550c2e: Downloading 3.784 MB/65.7 MB

8f229c550c2e: Downloading 23.25 MB/65.7 MB

f75a34586856: Download complete

8f229c550c2e: Pull complete

8e1fb71e8df6: Pull complete

f75a34586856: Pull complete

8744e322b832: Pull complete

d5165bfce78f: Pull complete

Digest: sha256:edf05697d8ea17028a69726b4b450ad48da8b29884cd640fec950c904bfb50ce

Status: Downloaded newer image for ubuntu:14.04

 ---&gt; 302fa07d8117

Step 2/4 : MAINTAINER birdben \(191654006@163.com\)

 ---&gt; Running in e2f4bb548ab2

 ---&gt; 26371304c9de

Removing intermediate container e2f4bb548ab2

Step 3/4 : RUN apt-get install -y openssh-server

 ---&gt; Running in de9068a3ba1c

Reading package lists...

Building dependency tree...

Reading state information...

E: Unable to locate package openssh-server

The command '/bin/sh -c apt-get install -y openssh-server' returned a non-zero code: 100

localhost:docker didi$

localhost:docker didi$

localhost:docker didi$

localhost:docker didi$ docker build -t docker-whale .

Sending build context to Docker daemon 2.048 kB

Step 1/4 : FROM ubuntu:14.04

 ---&gt; 302fa07d8117

Step 2/4 : MAINTAINER birdben \(191654006@163.com\)

 ---&gt; Using cache

 ---&gt; 26371304c9de

Step 3/4 : RUN apt-get install -y openssh-server

 ---&gt; Running in 5c9c6fec9b2d

Reading package lists...

Building dependency tree...

Reading state information...

E: Unable to locate package openssh-server

The command '/bin/sh -c apt-get install -y openssh-server' returned a non-zero code: 100

localhost:docker didi$ vi Dockerfile

localhost:docker didi$

localhost:docker didi$ docker build -t="birdben/ubuntu:v1" .

Sending build context to Docker daemon 2.048 kB

Step 1/4 : FROM ubuntu:14.04

 ---&gt; 302fa07d8117

Step 2/4 : MAINTAINER birdben \(191654006@163.com\)

 ---&gt; Using cache

 ---&gt; 26371304c9de

Step 3/4 : RUN apt-get install -y openssh-server

 ---&gt; Running in 691759c7f332

Reading package lists...

Building dependency tree...

Reading state information...

E: Unable to locate package openssh-server

The command '/bin/sh -c apt-get install -y openssh-server' returned a non-zero code: 100

localhost:docker didi$









