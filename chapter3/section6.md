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

