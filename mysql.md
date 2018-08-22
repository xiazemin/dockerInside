所使用服务器列表



tpl01	NAT	29. 159	mysql\(源码）	/etc/my.cnf	提供mysql服务（主）	yw009	数据库主从+代理搭建	mysql 5.6.39

tpl02	NAT	29. 152	mysql\(源码）	/etc/my.cnf	mysql服务（从）	yw009	数据库主从+代理搭建	mysql 5.6.39

tpl03	NAT	29. 160	mysql-proxy	/usr/local/mysql-proxy	代理服务	yw009	数据库主从+代理搭建	0.8.5

work	NAT	29. 158	\	\	mysql客户端	yw009	数据库主从+代理搭建	\

步骤一：安装mysql-proxy



1\)下载mysql-proxy 在github.com 上下载0.8.5版



16 tar -xvf mysql-proxy-rel-0.8.5.tar



18 yum -y install lua



54 yum install lua-devel



33 yum install cmake



34 yum install make



41 yum -y install gcc openssl-devel pcre-devel zlib-devel ncurses-devel



44 yum -y install libtool



63 yum -y install gcc gcc-c++



79 yum search flex



80 yum install flex.x86\_64



51 yum install mysql-devel



68 yum -y install glib2-devel



70 yum -y install libevent-devel



94 ./autogen.sh



96 ./configure --prefix=/usr/local/mysql-proxy



make



make install



79 yum search flex



80 yum install flex.x86\_64



126 mkdir /usr/local/mysql-proxy/lua



127 cp lib/rw-splitting.lua /usr/local/mysql-proxy/lua/



128 cp lib/admin-sql.lua /usr/local/mysql-proxy/lua/



131 cp -r lib/proxy /usr/local/mysql-proxy/lua/



132 cd /usr/local/mysql-proxy/



135 vim lua/rw-splitting.lua



2\) 搭建数据库主从 Master \(tpl01\) ,Slave \(tpl02\) ；可参考其他篇博客



3）启动mysql-proxy服务



139 bin/mysql-proxy -P 192.168.29.160:3306 -b 192.168.29.159:3306 -r 192.168.29.152:3306 -s lua/rw-splitting.lua &



启动后可确认监听状态：



netstat -anptu \| grep mysql;



为了每次开机后能够自动运行mysql-proxy,可以将相关操作写到/etc/rc.local配置文件内：



步骤二： 测试读写分离



1\) 在MySQL Master服务器上设置用户授权



以root 用户为例，允许其从192.168.29.0/24 网段的客户机远程访问。首先登入Master服务器添加下列授权：



mysql &gt; grant all on\*.\* to root@'192.168.29.%' identified by '123qwe';



因为此前已配置mysql库的主从同步，Slave上的root授权会自动更新：



2）从客户机work 访问MySQL数据库



注意连接的是mysql-proxy服务器，而并不是Master或 Slave:



测试数据库写入操作：



mysql &gt; create database proxydb;



mysql &gt; use proxydb;



mysql &gt; create table proxytb\( id int\(4\), host varchar\(48\)\);



mysql &gt; insert into proxytb values\(1, 'aa'\), \(2, 'bb'\);



mysql &gt; select \* from proxytb;



3\) 在 Master 和 Slave 确认新建的库，表



4） 观察MySQL 代理访问的网络连接



在 Master上可看到来自Slave 和proxy代理的网路连接：



netstat -anptu \| grep mysql



在Proxy代理上 可以看到与MySQL读，写服务器的网络连接：



netstat -anptu \| grep mysql



5\) 怎么才能确定读的数据是确实无疑 来自从数据库呢？



可以 使用 root 用户登入 slave 修改一条记录（这样主从数据库数据就不一致了，我们方便观察）



mysql&gt; update proxytb set host='ee' where id=2;



在 客户机work 上 连接mysql 查看数据：



mysql &gt; select \* from proxytb;

