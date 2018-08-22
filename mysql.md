二、mysql-proxy实现读写分离



1、安装mysql-proxy



实现读写分离是有lua脚本实现的，现在mysql-proxy里面已经集成，无需再安装



下载：http://dev.mysql.com/downloads/mysql-proxy/



 

 

 

 

 

Shell

 

1

2

tar zxvf mysql-proxy-0.8.3-linux-glibc2.3-x86-64bit.tar.gz

mv mysql-proxy-0.8.3-linux-glibc2.3-x86-64bit /usr/local/mysql-proxy

2、配置mysql-proxy，创建主配置文件



 

 

 

 

 

 

Shell

 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

cd /usr/local/mysql-proxy

mkdir lua \#创建脚本存放目录

mkdir logs \#创建日志目录

cp share/doc/mysql-proxy/rw-splitting.lua ./lua \#复制读写分离配置文件

cp share/doc/mysql-proxy/admin-sql.lua ./lua \#复制管理脚本

vi /etc/mysql-proxy.cnf   \#创建配置文件

\[mysql-proxy\]

user=root \#运行mysql-proxy用户

admin-username=proxy \#主从mysql共有的用户

admin-password=123.com \#用户的密码

proxy-address=192.168.0.204:4000 \#mysql-proxy运行ip和端口，不加端口，默认4040

proxy-read-only-backend-addresses=192.168.0.203 \#指定后端从slave读取数据

proxy-backend-addresses=192.168.0.202 \#指定后端主master写入数据

proxy-lua-script=/usr/local/mysql-proxy/lua/rw-splitting.lua \#指定读写分离配置文件位置

admin-lua-script=/usr/local/mysql-proxy/lua/admin-sql.lua \#指定管理脚本

log-file=/usr/local/mysql-proxy/logs/mysql-proxy.log \#日志位置

log-level=info \#定义log日志级别，由高到低分别有\(error\|warning\|info\|message\|debug\)

daemon=true    \#以守护进程方式运行

keepalive=true \#mysql-proxy崩溃时，尝试重启

保存退出！

chmod 660 /etc/mysql-porxy.cnf

3、修改读写分离配置文件



 

 

 

 

 

Shell

 

1

2

3

4

5

6

7

8

vi /usr/local/mysql-proxy/lua/rw-splitting.lua

if not proxy.global.config.rwsplit then

 proxy.global.config.rwsplit = {

  min\_idle\_connections = 1, \#默认超过4个连接数时，才开始读写分离，改为1

  max\_idle\_connections = 1, \#默认8，改为1

  is\_debug = false

 }

end

4、启动mysql-proxy



 

 

 

 

 

 

Shell

 

1

2

3

4

/usr/local/mysql-proxy/bin/mysql-proxy --defaults-file=/etc/mysql-proxy.cnf

netstat -tupln \| grep 4000 \#已经启动

tcp 0 0 192.168.0.204:4000 0.0.0.0:\* LISTEN 1264/mysql-proxy

关闭mysql-proxy使用：killall -9 mysql-proxy

5、测试读写分离



1&gt;.在主服务器创建proxy用户用于mysql-proxy使用，从服务器也会同步这个操作



 

 

 

 

 

 

Shell

 

1

mysql&gt; grant all on \*.\* to 'proxy'@'192.168.0.204' identified by '123.com';

2&gt;.使用客户端连接mysql-proxy



 

 

 

 

 

Shell

 

1

mysql -u proxy -h 192.168.0.204 -P 4000 -p123.com

创建数据库和表，这时的数据只写入主mysql，然后再同步从slave，可以先把slave的关了，看能不能写入，这里我就不测试了，下面测试下读的数据！



 

 

 

 

 

Shell

 

1

2

3

mysql&gt; create table user \(number INT\(10\),name VARCHAR\(255\)\);

mysql&gt; insert into test values\(01,'zhangsan'\);

mysql&gt; insert into user values\(02,'lisi'\);

3&gt;.登陆主从mysq查看新写入的数据如下，



 

 

 

 

 

Shell

 

1

2

3

4

5

6

7

8

9

mysql&gt; use test;

Database changed

mysql&gt; select \* from user;

+--------+----------+

\| number \| name \|

+--------+----------+

\| 1 \| zhangsan \|

\| 2 \| lisi \|

+--------+----------+

4&gt;.再登陆到mysql-proxy，查询数据，看出能正常查询



 

 

 

 

 

Shell

 

1

2

3

4

5

6

7

8

9

mysql -u proxy -h 192.168.0.204 -P 4000 -p123.com

mysql&gt; use test;

mysql&gt; select \* from user;

+--------+----------+

\| number \| name \|

+--------+----------+

\| 1 \| zhangsan \|

\| 2 \| lisi \|

+--------+----------+

5&gt;.登陆从服务器关闭mysql同步进程，这时再登陆mysql-proxy肯定会查询不出数据



 

 

 

 

 

Shell

 

1

slave stop；

6&gt;.登陆mysql-proxy查询数据，下面看来，能看到表，查询不出数据



 

 

 

 

 

Shell

 

1

2

3

4

5

6

7

8

9

10

mysql&gt; use test;

Database changed

mysql&gt; show tables;

+----------------+

\| Tables\_in\_test \|

+----------------+

\| user \|

+----------------+

mysql&gt; select \* from user;

ERROR 1146 \(42S02\): Table 'test.user' doesn't exist

配置成功！真正实现了读写分离的效果！



 

