# ambassador

hostB中app产生的数据需要实时写入hostA中的oracle数据库。也就是hostB中的docker container需要link hostA中的docker container。



为了解决这个问题，有两个解决方案：



方案一：



将hostA中的oracle container对外expose 1521\(我们假定此处对外expose 1521\)，然后在hostB中的app container中修改/etc/hosts文件，将hostA的IP添加到hosts文件中。

