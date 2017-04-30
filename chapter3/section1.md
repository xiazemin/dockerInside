# 创建带sshd服务的docker image

$ docker pull hub.c.163.com/liuinstein/sshd:latest

latest: Pulling from liuinstein/sshd

af49a5ceb2a5: Pull complete

8f9757b472e7: Pull complete

e931b117db38: Pull complete

47b5e16c0811: Pull complete

9332eaf1a55b: Pull complete

dd2d5605e3a6: Pull complete

15715c17ed7d: Pull complete

658207b1e77a: Pull complete

03910d367015: Pull complete

Digest: sha256:c0c649e98defe07d149e3c6413a3d0ab93462be0c8252eae2b69acde29844ff4

Status: Downloaded newer image for hub.c.163.com/liuinstein/sshd:latest

$ docker run -d -p 222:22 hub.c.163.com/liuinstein/sshd:latest /usr/sbin/sshd -D

060157246b69b439a894ac5a8e18cc26f89452e23f7cd97fbf8383e66e3b0b1b

$ ssh root@localhost -p 222

The authenticity of host '\[localhost\]:222 \(\[127.0.0.1\]:222\)' can't be established.

ECDSA key fingerprint is SHA256:yD5WCTsSi5t8QnRYfHphiuXX/FEtWnYLhjRNYRrz3k0.

Are you sure you want to continue connecting \(yes/no\)?

Warning: Permanently added '\[localhost\]:222' \(ECDSA\) to the list of known hosts.

root@localhost's password:

root@060157246b69:~\#



