# nginx镜像
$ docker run --name some-nginx -v hub.c.163.com/library/nginx -d nginx
docker: Error response from daemon: invalid volume spec "hub.c.163.com/library/nginx": invalid volume specification: 'hub.c.163.com/library/nginx': invalid mount config for type "volume": invalid mount path: 'hub.c.163.com/library/nginx' mount path must be absolute.
See 'docker run --help'.
 $ docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d hub.c.163.com/library/nginx
docker: Error response from daemon: Conflict. The container name "/some-nginx" is already in use by container db657988c748c921093344647cbe7557922ae90286895807837ffdc9483c455d. You have to remove (or rename) that container to be able to reuse that name..
See 'docker run --help'.
 $ docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
docker-whale                  latest              ca5555513a01        27 hours ago        277 MB
xiazemin/docker-whale         latest              ca5555513a01        27 hours ago        277 MB
<none>                        <none>              26371304c9de        28 hours ago        188 MB
hub.c.163.com/library/nginx   latest              46102226f2fd        3 days ago          109 MB
nginx                         latest              46102226f2fd        3 days ago          109 MB
ubuntu                        14.04               302fa07d8117        2 weeks ago         188 MB
hub.c.163.com/library/mysql   latest              d5127813070b        2 weeks ago         407 MB
docker/whalesay               latest              6b362a9f73eb        23 months ago       247 MB
 $  docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d hub.c.163.com/library/nginx
docker: Error response from daemon: Conflict. The container name "/some-nginx" is already in use by container db657988c748c921093344647cbe7557922ae90286895807837ffdc9483c455d. You have to remove (or rename) that container to be able to reuse that name..
See 'docker run --help'.
 $  docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d hub.c.163.com/library/nginx
docker: Error response from daemon: Conflict. The container name "/some-nginx" is already in use by container db657988c748c921093344647cbe7557922ae90286895807837ffdc9483c455d. You have to remove (or rename) that container to be able to reuse that name..
See 'docker run --help'.
 $
 $ docker run --name some-nginx -d some-content-nginx
Unable to find image 'some-content-nginx:latest' locally
docker: Error response from daemon: repository some-content-nginx not found: does not exist or no pull access.
See 'docker run --help'.
 $  docker run --name some-nginx -d -p 8080:80 hub.c.163.com/library/nginx
docker: Error response from daemon: Conflict. The container name "/some-nginx" is already in use by container db657988c748c921093344647cbe7557922ae90286895807837ffdc9483c455d. You have to remove (or rename) that container to be able to reuse that name..
See 'docker run --help'.
 $  docker run --name xiazemin-nginx -d -p 8080:80 hub.c.163.com/library/nginx
76470df045d73e5223570ad8a3f16a30f9d1f3a3aeb6e133d2746d05d17a3620
docker: Error response from daemon: driver failed programming external connectivity on endpoint xiazemin-nginx (60ad32239b48ee28569d6c2455ce72f866c02c5ef1e8895566331a689654721f): Error starting userland proxy: Bind for 0.0.0.0:8080 failed: port is already allocated.
 $  docker run --name xiazemin-nginx -d -p 8088:80 hub.c.163.com/library/nginx
docker: Error response from daemon: Conflict. The container name "/xiazemin-nginx" is already in use by container 76470df045d73e5223570ad8a3f16a30f9d1f3a3aeb6e133d2746d05d17a3620. You have to remove (or rename) that container to be able to reuse that name..
See 'docker run --help'.
 $  docker run --name xzm-nginx -d -p 8088:80 hub.c.163.com/library/nginx
46f62605a0ca8770ca4175bf98dd671245dee1cd48b8792d66d233934f82110e