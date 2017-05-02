# ambassador

hostB中app产生的数据需要实时写入hostA中的oracle数据库。也就是hostB中的docker container需要link hostA中的docker container。

为了解决这个问题，有两个解决方案：

方案一：

将hostA中的oracle container对外expose 1521\(我们假定此处对外expose 1521\)，然后在hostB中的app container中修改/etc/hosts文件，将hostA的IP添加到hosts文件中。

这种方案的优点就是可以根据实际情况自由配置," 自己的app掌控在自己手中 "。

但是缺点也很严重，首先每次run container时都需要修改hosts文件，而且每次host环境发生变化，都需要维护hosts文件，因此后续的维护成本很高。其次，如果遇到其他人开发的docker image，我们未必有权限来修改hosts文件。

所以此方案也仅仅用作开发测试使用，不推荐正式采用。

方案二：

Docker官方提供了一种ambassador的agent方案。此方案借助一个名为svendowideit/ambassador的image，将不同host进行解耦合。

![](/assets/importam.png)

具体实施步骤如下：



首先我们要在hostA和hostB之间pull svendowideit/ambassador。



1 docker pull svendowideit/ambassador

根据部署图可得知，是hostB要link hostA的container，因此我们需要先启动hostA的container\(此时可以理解hostA为server段，hostB为client段\)。



docker run  -d  --name oracle tirtool/oracle11g:latest

然后启动hostA中的ambassador container。



docker run  -d --link oracle:oracle -p 1521:1521 --name ambassador svendowideit/ambassador:latest

标红的部分是需要重点注意的地方，这个命令也就是将hostA中的oracle container直接暴露在ambassador之中，这样ambassador才能将访问请求转发到oracle container中。



此刻，hostA中的事情都已经处理完了，我们开始处理hostB。



与hostA的container启动顺序不用，我们需要先start hostB的ambassador。因为依据部署图可知，hostB中是app container调用ambassador container，所以需要先保证ambassador已经启动，才能启动app container。



docker run -d --name ambassador-oracle --expose 1521 -e ORACLE\_PORT\_1521\_TCP=tcp://&lt;&lt;hostA IP&gt;&gt;:1521 svendowideit/ambassador

标红的部分很重要，直接决定了hostB和hostA是否可以直接通讯。稍后，我们可以来解读一下这部分参数。



当hostB中的ambassador启动成功后，我们开始启动app container。



docker run --link ambassador-oracle:oracle  --name bw base/ubuntu:14.04

大家注意到我们在app container中将ambassador-oracle alias为oracle，这样在app container中的/etc/hosts文件中会出现一条记录：



10.1.0.3        oracle



此刻app container中app产生数据后，如果调用oracle:1521，那么首先将请求发往hostB的ambassador的1521端口。hostB的ambassador会将数据转发到hostA的1521端口。而此时hostA中的ambassador在listen 1521端口，接收到请求后会将数据转发至hostA的oracle container中。oracle container处理完毕后将response返回值ambassador，ambassador再依次回传。从而达到不同host中container相互访问的目的。



我们再看一下在hostB中执行  --expose 1521 -e ORACLE\_PORT\_1521\_TCP=tcp:// &lt;&lt;hostA IP&gt;&gt;:1521 时发生了什么事情。ambassador最重要的一项任务就是将hostB的1521端口同hostA的1521端口进行了端口映射。



其实执行的是下面的命令：



socat TCP4-LISTEN:1521,fork,reuseaddr TCP4:&lt;hostA IP&gt;:1521



这条命令采取fork模式，将本机的1521的数据转发到hostA的1521端口。而这条命令所需的参数来自于一个shell脚本：



env \| grep \_TCP= \| sed 's/.\*\_PORT\_\\(\[0-9\]\*\\)\_TCP=tcp:\/\/\\(.\*\\):\\(.\*\\)/socat TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/'  \| sh && top



这条命令在env中找寻所有\*\_TCP的环境变量,我们在启动时设定-e ORACLE\_PORT\_1521\_TCP=tcp://&lt;&lt;hostA IP&gt;&gt;:1521 所以可以找到ORACLE\_PORT\_1521\_TCP这条变量。找到后执行正则表达式"s/.\*\_PORT\_\\(\[0-9\]\*\\)\_TCP=tcp:\/\/\\(.\*\\):\\(.\*\\)/socat TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/" 替换后的shell command就是socat TCP4-LISTEN:1521,fork,reuseaddr TCP4:&lt;hostA IP&gt;:1521。



因此ambassador方案就是很巧妙的将不同host的port进行了桥接，而这些对docker使用者都是透明的。但这个方案也是有一些瑕疵的，就是如果新增container之后，需要重启或者新增ambassador，所以如果一个ambassador同时对应多个container，那么在维护上面就会稍许麻烦些，但维护成本比方案一低了很多。

